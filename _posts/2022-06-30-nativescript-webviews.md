# Call native methods from Webviews with the NativeScript ios runtime

NativeScript runtime allows calling native APIs without creating bridge classes or other boilerplate code. If we match this power to the concept of executing javascript code on a background thread (as React Native does) and reusing web components as UI, the level of code sharing may be as high as actually having your web application deployed both as such and as a mobile app.

Let me illustrate this concept - imagine having a responsive React or Angular (or whatever) web app that works as well on a mobile browser. You can create a PWA (progressive web app) but you still won't have access to push notifications and other native APIs. Other option is to pack it as a webview app and use the JavascriptCore API to hook to specific js calls in order to execute native code and then a callback. The obvious downside of this approach is that it lacks generallity - you will still have to call the native APIs from Objective-C or Swift. 

But if we can send data from the webview to the native world, then we can send javascript code and execute it with the NativeScript runtime. This can be used via a similar to the web workers API which receives messages and returns result. The following graph illustrates it:

![](https://raw.githubusercontent.com/tdermendjiev/tdermendjiev.github.io/master/assets/img/Screenshot%202022-06-30%20at%2014.58.01.png)

When the NativeScript runtime is loaded the `RunModule` method is called and what it does is basically calling the `require` function with the app's path as a parameter.

```
bool ModuleInternal::RunModule(Isolate* isolate, std::string path) {
    std::shared_ptr<Caches> cache = Caches::Get(isolate);
    Local<Context> context = cache->GetContext();
    Local<Object> globalObject = context->Global();
    Local<Value> requireObj;
    bool success = globalObject->Get(context, ToV8String(isolate, "require")).ToLocal(&requireObj);
    tns::Assert(success && requireObj->IsFunction(), isolate);
    Local<v8::Function> requireFunc = requireObj.As<v8::Function>();
    Local<Value> args[] = { ToV8String(isolate, path) };
    Local<Value> result;
    success = requireFunc->Call(context, globalObject, 1, args).ToLocal(&result);
    return success;
}
```

Our mission now is a bit different though - we want V8 to execute some js code and eventually return the result. This can be done via the `ScriptCompiler` class:

```
void ModuleInternal::RunScript(Isolate* isolate, std::string script) {
    std::shared_ptr<Caches> cache = Caches::Get(isolate);
    Local<Context> context = cache->GetContext();
    Local<Object> globalObject = context->Global();
    Local<Value> requireObj;
    bool success = globalObject->Get(context, ToV8String(isolate, "require")).ToLocal(&requireObj);
    tns::Assert(success && requireObj->IsFunction(), isolate);
    Local<Value> result;
    this->RunScriptString(isolate, context, script);
}
...
MaybeLocal<Value> ModuleInternal::RunScriptString(Isolate* isolate, Local<Context> context, const std::string scriptString) {
    ScriptCompiler::CompileOptions options = ScriptCompiler::kNoCompileOptions;
    ScriptCompiler::Source source(tns::ToV8String(isolate, scriptString));
    TryCatch tc(isolate);
    Local<Script> script = ScriptCompiler::Compile(context, &source, options).ToLocalChecked();
    MaybeLocal<Value> result = script->Run(context);
    return result;
}
```

In `Runtime.m` we will implement this:

```
void Runtime::RunScript(const std::string script) {
    Isolate* isolate = this->GetIsolate();
    v8::Locker locker(isolate);
    Isolate::Scope isolate_scope(isolate);
    HandleScope handle_scope(isolate);
    this->moduleInternal_->RunScript(isolate, script);
}
```

And then in `NativeScript.m`:

```
- (void)runScriptString: (NSString*) script runLoop: (BOOL) runLoop {

    std::string cppString = std::string([script UTF8String]);
    runtime_->RunScript(cppString);
    
    if (runLoop) {
        CFRunLoopRunInMode(kCFRunLoopDefaultMode, 0, true);
    }


    tns::Tasks::Drain();
}
```

Now that we can execute any javascript code from native, we need some way to communicate with the webview.

Let's start with receiving messages from the loaded page. If our viewcontroller conforms to the `WKScriptMessageHandler` protocol it will be able to receive messages sent via the `postMessage()` function. For example:

```
function sendMessage() {
    window.webkit.messageHandlers.executor.postMessage("console.log('Hello from NativeScript')")
}
```

In order to handle the message in Swift we have to implement `userController:didReceiveMessage` method in the webview delegate (e.g. the viewcontroller).

```
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    
    if (message.name == @"executor") {
        [_ns runScriptString:message.body runLoop:false];
    }
}
```

Now that we have received the code to be executed (`"console.log('Hello from NativeScript')"`) we pass it to the `runScriptString:runLoop` method and `'Hello from NativeScript!'` is printed in the console.

You can learn more about interacting between `WKWebView` and javascript from [this](https://dev.to/gualtierofr/wkwebview-and-javascript-interaction-1pbl) article.

Let's implement an interface for interacting with the NativeScript runtime. It is similar to the web worker's and would allow passing the code to be executed in the worker's constructor.

Start with something like this:

```
class NSWorker {

    constructor(script) {
        this.script = script;
        this.onerror = null;
        this.onmessage = null;
        this.onmessageerror = null;
        this.execute();
    }

    postMessage(msg) {
        if (window && window.webkit) {
            window.webkit.messageHandlers.postMessageListener.postMessage(msg);
        } else {
            console.log("No webkit")
        }
    }

    execute() {
        if (window && window.webkit) {
            window.webkit.messageHandlers.executor.postMessage(this.script);
        } else {
            console.log("No webkit")
        }
    }

}

let onNativeMessage = function(msg) {
    console.log("Message from native: " + msg)
}
```



This code should be executed when the page is loaded so we have the NSWorker class available to the page's context. This can be done via the `webView:didFinishNavigation:` method.

```
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    [_webview evaluateJavaScript:[self getWorkerScript] completionHandler:^(id _Nullable res, NSError * _Nullable error) {
        if (error) {
            NSLog([error debugDescription]);
        }
    }];
}

-(NSString*) getWorkerScript {
    return @"class NSWorker {\r\n\r\n    constructor(script) {\r\n        this.script = script;\r\n        this.onerror = null;\r\n        this.onmessage = null;\r\n        this.onmessageerror = null;\r\n        this.execute();\r\n    }\r\n\r\n    postMessage(msg) {\r\n        if (window && window.webkit) {\r\n            window.webkit.messageHandlers.postMessageListener.postMessage(msg);\r\n        } else {\r\n            console.log(\"No webkit\")\r\n        }\r\n    }\r\n\r\n    execute() {\r\n        if (window && window.webkit) {\r\n            window.webkit.messageHandlers.executor.postMessage(this.script);\r\n        } else {\r\n            console.log(\"No webkit\")\r\n        }\r\n    }\r\n\r\n    terminate() {\r\n        if (window && window.webkit) {\r\n            window.webkit.messageHandlers.terminator.postMessage(this.script);\r\n        } else {\r\n            console.log(\"No webkit\")\r\n        }\r\n    }\r\n\r\n}\r\n\r\nlet onNativeMessage = function(msg) {\r\n    console.log(\"Message from native: \" + msg)\r\n}";
}
```
Now in our webpage we can initialize the worker

When the message is correctly parsed and executed by the runtime the result should be passed back to the web page. For that purpose I defined a global `postmessage` function in the v8 runtime:

```
void Runtime::DefineGlobalPostMessage(Isolate* isolate, Local<ObjectTemplate> globalTemplate) {
    Local<FunctionTemplate> postMessageTemplate = FunctionTemplate::New(isolate, Runtime::PostMessageToMainCallback);

    Local<v8::String> postMessageName = ToV8String(isolate, "postmessage");
    globalTemplate->Set(postMessageName, postMessageTemplate);
}

void Runtime::PostMessageToMainCallback(const FunctionCallbackInfo<Value>& info) {
    // Send message from worker to main
    Isolate* isolate = info.GetIsolate();

    try {
        if (info.Length() < 1) {
            throw NativeScriptException("Not enough arguments.");
        }

        if (info.Length() > 1) {
            throw NativeScriptException("Too many arguments passed.");
        }


        Local<Value> error;
        Local<Value> result = Worker::Serialize(isolate, info[0], error);
        if (result.IsEmpty()) {
            isolate->ThrowException(error);
            return;
        }

        std::string message = tns::ToString(isolate, result);
        
        [[WebviewDispatcher shared] postMessage: [NSString stringWithCString:message.c_str()
                                                                    encoding:[NSString defaultCStringEncoding]]];
        
    } catch (NativeScriptException& ex) {
        ex.ReThrowToV8(isolate);
    }
}
```

The `WebviewDispatcher` class is used only for passing messages between the NativeScript runtime and the web page. Currently it's just sending the `postmessage` result via the `NSNotificationCenter` and the notification is handled in the viewcontroller:

```
- (void) messagePosted:(NSNotification*)notification {
    if ([notification.object isKindOfClass:[NSString class]]) {
        [_webview evaluateJavaScript:[NSString stringWithFormat:@"onNativeMessage(%@)", notification.object] completionHandler:NULL];
    }

}
```

You probably noticed that in the webpage's js code we override the `onNativeMessage` function. Now it will receive the message passed from the native world and the webpage will receive the json object - `{"data":"iPhone"}`.




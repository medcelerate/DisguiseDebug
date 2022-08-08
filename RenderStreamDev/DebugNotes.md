# Debugging RenderStream

## Debugging Plugins

You will always need to launch your renderstream addon from disguise. However there are a few ways to be able to work with it after it launches.

Adding this code to your main function will allow you to wait for the debugger to attach. 

```cpp
        while (!::IsDebuggerPresent()) {
            rs_logToD3("Here");
            ::Sleep(100);
        }
```

You can attach Visual Studio Debugger by clicking **Debug -> Attach To Process** or `ctrl+alt+P`

## RenderStream Code

In the examples provided by Disguise the following macro is used to load functions into variables initialized in the main scope.
```cpp
#define LOAD_FN(FUNC_NAME) \
        FUNC_NAME = reinterpret_cast<decltype(FUNC_NAME)>(GetProcAddress(RSLib, #FUNC_NAME)); \
    if (!FUNC_NAME) { \
        ofLogError("ofxRenderStream") << "Failed to get function " #FUNC_NAME " from DLL" << std::endl; \
        return 2; \
```
Its important to note that this will scope the functions to simply the main function. If using it as part of a class you an still use the macro but it will require something similar as below.
```cpp
class Test {
    protected:
		decltype(rs_initialise)* rs_initialise;
		decltype(rs_initialiseGpGpuWithoutInterop)* rs_initialiseGpGpuWithoutInterop;
		decltype(rs_getStreams)* rs_getStreams;
		decltype(rs_awaitFrameData)* rs_awaitFrameData;
		decltype(rs_getFrameCamera)* rs_getFrameCamera;
		decltype(rs_sendFrame)* rs_sendFrame;
		decltype(rs_shutdown)* rs_shutdown;
		decltype(rs_logToD3)* rs_logToD3;
}
```

### Smart Pointers

It may be possible to do the below in order to make the pointers to the functions smart pointers.

```cpp
std::shared_ptr<decltype(rs_initialize)> rs_initialise;
```

And use this macro
```cpp
#define LOAD_FN(FUNC_NAME) \
        FUNC_NAME = std::shared_ptr<decltype(rs_initialize)>(reinterpret_cast<decltype(FUNC_NAME)>(GetProcAddress(RSLib, #FUNC_NAME))); \
    if (!FUNC_NAME) { \
        ofLogError("ofxRenderStream") << "Failed to get function " #FUNC_NAME " from DLL" << std::endl; \
        return 2; \
```

### Type conversions of pixel data.

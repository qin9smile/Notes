In Asynchronous Javascript, we have a callback function, an event loop, and a
task queue. The callback function is acted upon by the call stack during
execution after the call abck function has been pushed to the stack by the
event loop.

### Call Stack 调用栈

temporarily store and manage function invocation.

`Temporarily store`: When a function is invoked, the function, its parameters,
and variables are pushed into the call stack to form a `stack frame`. This
stack frame is a memory location in the stack. The memory is cleared when the 
function returns as it is pop out of the stack.

`Manage function invocation`: The call stack maintains a record of the position 
of each stack frame. It knows the next function to be executed (and will remove 
it after execution). This is what makes code execution in Javascript synchronous.

`Stack Overflow`: when there is a recursive function without an exit point or 
before exit call too many times. The browser has an maximum stack call that it 
can accommodate before throwing a stack error.

setTimeout() / setInterval() is in milliseconds not microseconds

### 执行上下文

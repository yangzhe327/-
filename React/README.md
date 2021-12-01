

### Home
[Home](https://github.com/yangzhe327/Front-end-Study-Notes)

# useState
  useState的初始值如果是个执行函数，每次render的时候都会重新调用这个执行函数，但并不进行赋值
  ```
  const [state, setState] = useState(func())
  ```
  声明函数只会在第一次render的时候调用并且赋值
    ```
  const [state, setState] = useState(() => func())
  ```

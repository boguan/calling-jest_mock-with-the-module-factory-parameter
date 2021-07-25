## 模拟函数时的注意点

借用官网的例子：

```javascript
import SoundPlayer from './sound-player';
const mockPlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: mockPlaySoundFile};
  });
});
```

然而，在测试该示例时，我发现它是有局限性的。定义函数jest.fn()返回值得变量只能是 mock 开头的。

如果我们将 mockPlaySoundFile 改成 fakePlaySoundFile 

```javascript
// Note: this will fail
import SoundPlayer from './sound-player';
const fakePlaySoundFile = jest.fn();
jest.mock('./sound-player', () => {
  return jest.fn().mockImplementation(() => {
    return {playSoundFile: fakePlaySoundFile};
  });
});
```

会报如下错误：

```shell
The module factory of `jest.mock()` is not allowed to reference any out-of-scope variables.
```

这是由于[变量提升](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)导致的。Jest docs 中[声明](https://jestjs.io/docs/es6-class-mocks#calling-jestmock-with-the-module-factory-parameter)：

*A limitation with the factory parameter is that, since calls to jest.mock() are hoisted to the top of the file, it's not possible to first define a variable and then use it in the factory. An exception is made for variables that start with the word 'mock'. It's up to you to guarantee that they will be initialized on time!*

实际被测试文件内的执行顺序为：
1. jest.mock()s
2. imports
3. 在导入（import）的测试对象内部使用 Jest 模拟
4. 在测试文件中声明变量 mockPlaySoundFile 
5. 执行测试和调用测试对象


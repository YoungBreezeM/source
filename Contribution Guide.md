# 贡献指南


首先，任何形式的贡献都将受到高度赞赏，您不必成为C ++的专业人士，我也不是。如果您是使用nan开发原生Node.js的新手，并且希望开始使用它，则可以 快速浏览一下我的文章系列：[**使用C ++的本机Node.js模块的教程**]（https://medium.com/netscape/tutorial-building-native-c-modules-for- node-js-using-nan-part-1-755b07389c7c）

大多数时候，添加绑定的方式与代码库中已经存在的方式类似。 因此，您可以以现有的东西为例来帮助您入门。 在下面的内容中，您可以找到向软件包添加新的OpenCV函数绑定的一些基本准则。

## API设计

API的设计使得

1：传递给函数调用的参数经过类型检查，并在发生错误的情况下向用户显示适当的消息。 没有人希望通过巧合将垃圾传递给函数以使其静默失败，而这可能会产生意想不到的结果。

2：一个函数，它需要多个默认值的参数，可以通过传递带有命名参数的JSON对象代替可选参数来方便地调用该函数。

3：如果函数的第一个参数对应于OpenCV类中的一个（通常为cv :: Mat），则应将函数绑定导出为全局cv方法和包装类的类方法，以允许链接 函数调用，例如：`mat.resizeToMax（500）.toGray（）。mean（）`。

例如，考虑以下来自OpenCV 3官方文档的函数签名：

```c++
void GaussianBlur(InputArray src, OutputArray dst, Size ksize, double sigmaX, double sigmaY=0, int borderType=BORDER_DEFAULT)
```

cv :: InputArray通常对应于cv :: Mat，但有时它们也可以是点或向量的std :: vector。 所有OutputArrays都将转换为绑定中的返回值。 如果一个函数具有多个返回值，则将它们作为JSON对象返回，其中包含返回值作为键值对。

将可选参数作为命名参数传递将提供以下便利：能够传递单个可选参数而不必传递所有其他可选参数。

该函数应通过以下方式调用：

```javascript
const mat = new cv.Mat(...)

// required arguments
const size = new cv.Size(...)
const sigmaX = 1.2

// optional arguments
const sigmaY = 1.2
const borderType = cv.BORDER_CONSTANT

let dst

/* invocation on the global cv object: */

// with required arguments
dst = cv.gaussianBlur(mat, size, sigmaX)

// with optional arguments
dst = cv.gaussianBlur(mat, size, sigmaX, sigmaY)
dst = cv.gaussianBlur(mat, size, sigmaX, sigmaY, borderType)

// with named optional arguments as JSON object
dst = cv.gaussianBlur(mat, size, sigmaX, { sigmaY: 1.2 })
dst = cv.gaussianBlur(mat, size, sigmaX, { borderType: cv.BORDER_CONSTANT })
dst = cv.gaussianBlur(mat, size, sigmaX, { sigmaY: 1.2, borderType: cv.BORDER_CONSTANT })

/* invocation as a class method: */

// with required arguments
dst = mat.gaussianBlur(size, sigmaX)

// with optional arguments
dst = mat.gaussianBlur(size, sigmaX, sigmaY)
dst = mat.gaussianBlur(size, sigmaX, sigmaY, borderType)

// with named optional arguments as JSON object
dst = mat.gaussianBlur(size, sigmaX, { sigmaY: 1.2 })
dst = mat.gaussianBlur(size, sigmaX, { borderType: cv.BORDER_CONSTANT })
dst = mat.gaussianBlur(size, sigmaX, { sigmaY: 1.2, borderType: cv.BORDER_CONSTANT })
```

## 添加新功能绑定的指南

通过3个简单的步骤即可将新的nodejs绑定添加到OpenCV函数：

1.添加功能绑定
2.编写单元测试
3.添加TypeScript的类型声明

### 1. Add the Function Binding

Let's consider the GaussianBlur example. Since the first argument is a cv::Mat, we are going to make this a class method binding. Furthermore, GaussianBlur is implemented in the imgproc package of OpenCV (as we have to include `#include ` to use this method). Therefore we want to implement the binding in imgprocBindings.h:

```c++
namespace ImgprocBindings {

  ...

  class GaussianBlur : public CvClassMethodBinding<Mat> {
  public:
    void createBinding(std::shared_ptr<FF::Value<cv::Mat>> self) {
      // required parameters
      auto kSize = req<Size::Converter>();
      auto sigmaX = req<FF::DoubleConverter>();

      // optional parameters
      auto sigmaY = opt<FF::DoubleConverter>("sigmaY", 0);
      auto borderType = opt<FF::IntConverter>("borderType", cv::BORDER_CONSTANT);

      // return values
      auto blurMat = ret<Mat::Converter>("blurMat");

      // the actual function call
      executeBinding = [=]() {
        cv::GaussianBlur(self, blurMat->ref(), kSize->ref(), sigmaX->ref(), sigmaY->ref(), borderType->ref());
      };
    };
  };
}
```

To expose the synchronous and asynchronous bindings for GaussianBlur we first declare the global methods in Imgproc.h:

```c++
class Imgproc {

  ...

  static NAN_METHOD(GaussianBlur);
  static NAN_METHOD(GaussianBlurAsync);

}
```

And then expose the bindings in Imgproc.cc:

```c++
// in the init hook, we are telling the package to expose the
// global function bindings to the module object (target)
NAN_MODULE_INIT(Imgproc::Init) {

  ...

  Nan::SetMethod(target, "gaussianBlur", GaussianBlur);
  Nan::SetMethod(target, "gaussianBlurAsync", GaussianBlurAsync);
}

// synchronous binding
NAN_METHOD(Imgproc::GaussianBlur) {
  FF::syncBinding<ImgprocBindings::GaussianBlur>("Imgproc", "GaussianBlur", info);
}

// asynchronous binding
NAN_METHOD(Imgproc::GaussianBlurAsync) {
  FF::asyncBinding<ImgprocBindings::GaussianBlur>("Imgproc", "GaussianBlur", info);
}
```

We repeat this procedure for the class method bindings and declare them in MatImgproc.h

```c++
class MatImgproc {

  ...

  static NAN_METHOD(GaussianBlur);
  static NAN_METHOD(GaussianBlurAsync);

}
```

And then expose the bindings in MatImgproc.cc:

```c++
// in the init hook, we are telling the package to expose those bindings on the
// Mat prototype so that we can actually call it from JavaScript
void MatImgproc::Init(v8::Local<v8::FunctionTemplate> ctor) {

  ...

  Nan::SetPrototypeMethod(ctor, "gaussianBlur", GaussianBlur);
  Nan::SetPrototypeMethod(ctor, "gaussianBlurAsync", GaussianBlurAsync);
}

// synchronous binding
NAN_METHOD(MatImgproc::GaussianBlur) {
  Mat::syncBinding<ImgprocBindings::GaussianBlur>("GaussianBlur", info);
}

// asynchronous binding
NAN_METHOD(MatImgproc::GaussianBlurAsync) {
  Mat::asyncBinding<ImgprocBindings::GaussianBlur>("GaussianBlur", info);
}
```

### 2. Writing Unit Tests

We test the bindings directly from JS with a classic mocha + chai setup. The purpose of unit testing is not to ensure correct behaviour of OpenCV function calls as OpenCV functionality is tested and none of our business. However, we want to ensure that our bindings can be called without crashing, that all parameters are passed and objects unwrapped correctly and that the function call returns what we expect it to.

You can use generateAPITests to easily generate default tests for a function binding that is implemented sync and async. This will generate the tests which ensure that the synchronous as well as the callbacked and promisified async bindings are called correctly. However, you are welcome to write additional tests. For the gaussianBlur example we use generateClassMethodTests instead of generateAPITests, which will generate tests for the global method binding `cv.gaussianBlur(mat, ...)` as well as the class method binding `mat.gaussianBlur(...)`.

For the gaussianBlur example generating unit tests can by adding the following to imgprocTests.js located in test/tests/core/Mat:

```javascript
describe('gaussianBlur', () => {
  const matData = [
    [0, 0, 128],
    [0, 128, 255],
    [128, 255, 255]
  ]
  const mat = new cv.Mat(matData, cv.CV_8U)

  const expectOutput = (blurred) => {
    assertMetaData(blurred)(mat.rows, mat.cols, mat.type);
    expect(dangerousDeepEquals(blurred.getDataAsArray(), matData)).to.be.false;
  };

  const kSize = new cv.Size(3, 3);
  const sigmaX = 1.2;

  generateClassMethodTests({
    getClassInstance: () => mat,
    methodName: 'gaussianBlur',
    classNameSpace: 'Mat',
    methodNameSpace: 'Imgproc',
    getRequiredArgs: () => ([
      kSize,
      sigmaX
    ]),
    getOptionalArgsMap: () => ([
      ['sigmaY', 1.2],
      ['borderType', cv.BORDER_CONSTANT]
    ]),
    expectOutput
  });
});
```

### 3. Adding the Type Declaration

All type declarations are located in lib/typings. We simply add the type information of the signature of the function binding we just implemented to Mat.d.ts:

```typescript
export class Mat {

  ...

  gaussianBlur(kSize: Size, sigmaX: number, sigmaY?: number, borderType?: number): Mat;
  gaussianBlurAsync(kSize: Size, sigmaX: number, sigmaY?: number, borderType?: number): Promise<Mat>;
}
```

And we add the typings for the global function binding to cv.d.ts:

```typescript
export function gaussianBlur(mat: Mat, kSize: Size, sigmaX: number, sigmaY?: number, borderType?: number): Mat;
export function gaussianBlurAsync(mat: Mat, kSize: Size, sigmaX: number, sigmaY?: number, borderType?: number): Promise<Mat>;
```

And that's it! You can now open a Pull Request, which will be built on the CI.

## Implementing a Wrapped Class

If you want to add a new class wrapper for one of the OpenCV classes you should make this class extend the FF::ObjectWrap<TClass, T> template, which allows us to use all the helper functions and converters for the class:

```c++
// the second template parameter corresponds to the OpenCV class (cv::Mat)
class Mat : public FF::ObjectWrap<Mat, cv::Mat> {
public:
  // declare the constructor, required for FF::ObjectWrap
  static Nan::Persistent<v8::FunctionTemplate> constructor;

  // declare the class name, required for FF::ObjectWrap
  static const char* getClassName() {
    return "Mat";
  }

  // every class binding has to implement and Init method
  static NAN_MODULE_INIT(Init);
};
```

I provided a step by step guide for implementing class bindings in this [tutorial](https://medium.com/netscape/tutorial-building-native-c-modules-for-node-js-using-nan-part-1-755b07389c7c).

## CI

For continous integration we use AppVeyor and Travis CI, which will run a rebuild of the package on Windows and Linux and run the unit tests for each of the maintained OpenCV versions. This ensures compatibility across the OpenCV versions as in some minor cases the OpenCV interface may have changed or new features have been added.

The build task will be executed on every push to your working branch as well as every pull request before merging to the master branch. If you have docker set up on your local machine you can run the build tasks on your local machine via the provided npm scripts under ci/test. For example to execute a build for OpenCV 3.4.6 under node 11:

```bash
# with contrib (OpenCV extra modules)
npm run test 3.4.6-contrib 11

# without contrib
npm run test 3.4.6 11
```


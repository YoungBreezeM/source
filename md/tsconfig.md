```
{
    ``"compileOnSave": false,
    ``"compilerOptions": {
        ``"outDir": "./dist/out-tsc",//输出目录
        ``"baseUrl": "src",
        ``"sourceMap": true,//把 ts 文件编译成 js 文件的时候，同时生成对应的 map 文件
        ``"removeComments": true,//编译 js 的时候，删除掉注释
        ``"declaration": false,
        ``"moduleResolution": "node",//模块的解析
        ``"emitDecoratorMetadata": true,//给源码里的装饰器声明加上设计类型元数据。查看issue #2577了解更多信息。
        ``"experimentalDecorators": true,//启用实验性的ES装饰器。
        ``"target": "es5",//编译目标平台
        ``//exclude：不包含的编译目录
        ``//noImplicitAny：true/false；为 false 时，如果编译器无法根据变量的使用来判断类型时，将用 any 类型代替。为 true 时，进行强类型检查，会报错
        ``"typeRoots": [
        ``"node_modules/@types"
        ``],
        ``"lib": [//添加需要的解析的语法，否则TS会检测出错。
        ``"es2016",
        ``"dom"
        ``]
      ``}
}
```
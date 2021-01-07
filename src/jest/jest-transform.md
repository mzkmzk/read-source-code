# jest-transform

未完结

类图: https://www.processon.com/diagraming/5ff56ca26376891d8b385cc7

# 辅助方法说明

### calcIgnorePatternRegExp 

作用: 获取忽略transform的正则表达式

参数: `Config.ProjectConfig`

配置方法见 [config里transformIgnorePatterns的配置方法](https://jestjs.io/docs/en/configuration#transformignorepatterns-arraystring)

```
transformIgnorePatterns [array<string>]
Default: ["/node_modules/", "\\.pnp\\.[^\\\/]+$"]
```

这里主要运用到正则里`|`的语法, 让transformIgnorePatterns里的数组都可以在一个正则表达出来, 而不用写一个for循环一次次去match正则

```javascript
const calcIgnorePatternRegExp = (config: Config.ProjectConfig) => {
  if (
    !config.transformIgnorePatterns ||
    config.transformIgnorePatterns.length === 0
  ) {
    return undefined;
  }

  return new RegExp(config.transformIgnorePatterns.join('|'));
};
```

### calcTransformRegExp

作用: 计算config中transform属性最终对应的正则

参数: `Config.ProjectConfig`

配置方法详见: [config.transform配置](https://jestjs.io/docs/en/configuration#transform-objectstring-pathtotransformer--pathtotransformer-object)

例如`jest.config.js`中配置的值是

```javascript
transform: {
    '.*\\.(js)$': 'babel-jest',
    '.*\\.(vue)$': 'vue-jest'
}
```

config.transform会被其他模块转换为

```javascript
transform: [
    [
        '.*\\.(js)$',
        '/Users/maizhikun/project/10800-xxx-pay-xxx-com-pages/node_modules/babel-jest/build/index.js',
        {} //传递给babel-jest的配置参数
    ],
    [
        '.*\\.(vue)$',
        '/Users/maizhikun/project/10800-xxx-pay-xxx-com-pages/node_modules/vue-jest/vue-jest.js',
        {} //传递给vue-jest的配置参数
    ]
],
```

所以`calcTransformRegExp`的方法返回只是将`'.*\\.(js)$'`这些转换正则表达式

源码: 

```javascript
const calcTransformRegExp = (config: Config.ProjectConfig) => {
  if (!config.transform.length) {
    return undefined;
  }

  const transformRegexp: Array<[RegExp, string, Record<string, unknown>]> = [];
  for (let i = 0; i < config.transform.length; i++) {
    transformRegexp.push([
      new RegExp(config.transform[i][0]),
      config.transform[i][1],
      config.transform[i][2],
    ]);
  }

  return transformRegexp;
};
```

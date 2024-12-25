# Knockout.js 构建工具

本文介绍如何使用各种构建工具来优化 Knockout.js 应用。

## Webpack 配置

### 基本配置

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader'
                }
            }
        ]
    }
};
```

### 优化配置

```javascript
// webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const TerserPlugin = require('terser-webpack-plugin');

module.exports = merge(common, {
    mode: 'production',
    optimization: {
        minimizer: [new TerserPlugin()],
        splitChunks: {
            chunks: 'all'
        }
    }
});
```

## Rollup 配置

### 基本配置

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'iife'
    },
    plugins: [
        resolve(),
        commonjs()
    ]
};
```

### 生产环境配置

```javascript
// rollup.config.prod.js
import { terser } from 'rollup-plugin-terser';

export default {
    // ... 基本配置
    plugins: [
        // ... 其他插件
        terser()
    ]
};
```

## Gulp 任务

### 基本任务

```javascript
// gulpfile.js
const gulp = require('gulp');
const concat = require('gulp-concat');
const uglify = require('gulp-uglify');

gulp.task('scripts', function() {
    return gulp.src('src/**/*.js')
        .pipe(concat('all.js'))
        .pipe(uglify())
        .pipe(gulp.dest('dist'));
});
```

### 监视文件变化

```javascript
gulp.task('watch', function() {
    gulp.watch('src/**/*.js', gulp.series('scripts'));
});
```

## NPM 脚本

```json
{
    "scripts": {
        "build": "webpack --config webpack.prod.js",
        "dev": "webpack serve --config webpack.dev.js",
        "lint": "eslint src",
        "test": "jest"
    }
}
```

## 构建优化

### 1. 代码分割

```javascript
// webpack.config.js
module.exports = {
    optimization: {
        splitChunks: {
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                }
            }
        }
    }
};
```

### 2. 懒加载

```javascript
// 组件懒加载
function loadComponent() {
    return import('./components/myComponent')
        .then(module => {
            ko.components.register('my-component', module.default);
        });
}
```

### 3. Tree Shaking

```javascript
// package.json
{
    "sideEffects": false
}
```

## 开发环境配置

### 1. 热重载

```javascript
// webpack.dev.js
module.exports = {
    devServer: {
        hot: true,
        open: true
    }
};
```

### 2. Source Maps

```javascript
// webpack.dev.js
module.exports = {
    devtool: 'eval-source-map'
};
```

## 部署配置

### 1. 环境变量

```javascript
// config.js
const config = {
    apiUrl: process.env.API_URL || 'http://localhost:3000',
    debug: process.env.NODE_ENV !== 'production'
};
```

### 2. 静态资源处理

```javascript
// webpack.config.js
const CopyPlugin = require('copy-webpack-plugin');

module.exports = {
    plugins: [
        new CopyPlugin({
            patterns: [
                { from: 'public', to: 'dist' }
            ]
        })
    ]
};
```

## 性能优化

### 1. 代码压缩

```javascript
// webpack.prod.js
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
    plugins: [
        new CompressionPlugin()
    ]
};
```

### 2. 缓存优化

```javascript
// webpack.config.js
module.exports = {
    output: {
        filename: '[name].[contenthash].js'
    }
};
```

## 测试配置

### 1. Jest 配置

```javascript
// jest.config.js
module.exports = {
    transform: {
        '^.+\\.js$': 'babel-jest'
    },
    moduleNameMapper: {
        '\\.(css|less)$': 'identity-obj-proxy'
    }
};
```

### 2. Karma 配置

```javascript
// karma.conf.js
module.exports = function(config) {
    config.set({
        frameworks: ['jasmine'],
        files: [
            'src/**/*.js',
            'test/**/*.js'
        ]
    });
};
```

## 持续集成

### 1. GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Install
      run: npm install
    - name: Test
      run: npm test
    - name: Build
      run: npm run build
```

### 2. Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
    }
}
```

## 最佳实践

1. **构建流程优化**
   - 使用合适的构建工具
   - 配置合理的优化策略
   - 实施持续集成

2. **开发体验**
   - 配置热重载
   - 使用 Source Maps
   - 优化编译速度

3. **部署策略**
   - 实施自动化部署
   - 配置环境变量
   - 优化静态资源

## 常见问题

1. **构建速度慢**
   - 使用 cache-loader
   - 优化 loader 配置
   - 使用 DLL 插件

2. **打包体积大**
   - 实施代码分割
   - 启用 Tree Shaking
   - 优化依赖引入

3. **环境配置问题**
   - 检查环境变量
   - 验证配置文件
   - 确认构建命令

## 总结

构建工具的关键点：

1. 选择合适的构建工具
2. 优化构建配置
3. 实施自动化流程
4. 保证代码质量
5. 优化部署策略 
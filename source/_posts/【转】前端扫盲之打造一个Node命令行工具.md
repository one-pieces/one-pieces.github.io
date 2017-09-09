---
title: 【转】前端扫盲之打造一个Node命令行工具
tags: [Node, 前端, 命令行工具]
---
[原文链接](http://www.imooc.com/article/3384)

Node 给前端开发带来了很大的改变，促进了前端开发的自动化，我们可以简化开发工作，然后利用各种工具包生成生产环境。如运行 `sass src/sass/main.scss dist/css/main.css` 即可编译Sass文件。在实际的开发过程中，我们可能会有自己的特定需求，那么我们得学会如何创建一个Node命令行工具。

命令行接口：Command Line Interface，简称CLI，是Node提供的一个用于命令行交互的工具，本质是基于Node引擎运行的。

在前面的文章[前端扫盲-之打造一个自动化的前端项目](https://www.awesomes.cn/source/9)中，给大家留了一个问题，就是如何通过执行一条命令就生成我们需要的项目结构。今天我就带着大家一步一步解答这个问题。

我们的初步设想是，在指定目录下执行一个命令（假设为autogo）
```bash
autogo demo
```
就会生成一个目录名为`demo`的项目，里面包含有我们所需的基础文件结构。

## 开始

1.首先我们创建一个程序包清单（`package.json`文件）包含了该命令包的相关信息：
```bash
npm init
```
然后根据提示输入对应的信息，也可以一路回车，待生成好`package.json`文件后再作修改。

2.创建一个用于运行命令的脚本`bin/autogo.js`：
```js
#! /usr/bin/env node
console.log('hello')
```
然后我们执行
```bash
node bin/autogo.js
```
能够看到输出了`hello`，当然这不是我们想要的结果， 我们是要直接运行`autogo`命令就能输出`hello`。

3.告诉`npm`你的命令脚本文件是哪一个，这里我们需要给`package.json`添加一个`bin`字段：
```json
{
  "bin": {
    "autogo": "./bin/autogo.js"
  }
}
```
这里我们指定`autogo`命令的执行文件为`./bin/autogo.js`.

4.启用命令行：
```bash
npm link
```
这里我们通过`npm link`在本地安装了这个包用于测试，然后就可以通过
```bash
autogo
```
来运行命令了。

### 可能遇到的问题

1.有的同学可能在执行`autogo`命令后报下面的错误：
```bash
-bash: /usr/local/bin/autogo: /usr/local/bin/node^M: bad interpreter: No such file or directory
```
之所以出现这个错误是因为`bin/autogo.js`文件是在windows下做的编辑，windows下默认的换行是`\n\r`，而linux下默认的换行是`\n`，所以文件后的`\r`在linux下是不会被识别的，显示成了^M。

要解决这个问题的办法就是改变文件的编码，这里我们需要用到`dos2unix`这个包。

首先安装
```bash
sudo apt-get install dos2unix
```
然后
```bash
sudo dos2unix bin.autogo.js 
```
问题就解决了。

2.还有的同学可能会遇到下面这个报错
```bash
: No such file or directory
```
报这个错是因为`#! /usr/bin/env/node`没能识别出你的`node`的路径，需要将你的`node`安装路径（如`/usr/local/bin/`）加入到系统的PATH中。

其实你可以在测试环境中将这个标识换成`#! /usr/local/bin/node`（这里的`which node`找到你自己的`node`路径），再运行就没问题了。但是我们之所以用`#! /usr/bin/env/node`是因为这可以动态检测出不同用户各自的`node`路径，而不是写死的，毕竟不是所有用户的`node`命令都是在`/usr/local/bin/`下。得确保在发布之前将其改回成`#! /usr/bin/env/node`。

到此，一个本地的`npm`命令行工具就已经成功完成了，（可参见[官方文档](http://blog.npmjs.org/post/118810260230/building-a-simple-command-line-tool-with-npm)）接下来我们就来完善具体的功能。

## 创建项目结构

我们需要的项目结构大致如下，包含了所需的文件和文件夹（[详见](https://github.com/bimohxh/autogo/tree/master/structure)）。

要创建上面的结构，我们可以通过程序来创建多个文件和文件夹，但是对于这么多文件，而且每个文件里或许还有更多内容，所以我们应该用一个更简便的方法。

实际上我们可以先创建一个完整的结构，然后在执行命令时，通过程序把这些文件和文件夹整个复制到目标项目文件夹中去，最后再对某些文件做一些修改即可。

按照这个思路，我们根据上面的结构，将这些文件和文件夹创建到`structure`下，然后我们创建一个生成结构的方法`lib/generateStructure.js`（这里我们将功能模块放在`lib/`目录下）

```js
var Promise = require('bluebird'),
    fs = Promise.promisifyAll(require('fs-extra'));

function generateStructure(project) {
  return fs.copyAsync('structure', project, { clobber: true })
  .then(function(err) {
    if (err) return console.error(err);
  });
}

module.exports = generateStructure;
```
上面的代码就是通过`fs-extra`这个包（[查看文档](https://www.npmjs.com/package/fs-extra)）将`structure`目录下的内容复制到`project`参数的目标文件夹中。`fs-extra`是对`fs`包的一个扩展，方便我们对文件的操作。

这里我们用到了`bluebrid`（[查看文档](https://www.npmjs.com/package/bluebird)），这是一个实现Promise的库，因为这里牵涉到了对文件的操作，所以会有异步方法，而Promise就是专门解决这些异步操作嵌套回调的，能将其扁平化。

自然，我们应该安装这两个包：
```bash
npm install bluebird --save
npm install fs-extra --save
```

这里加上`--save`参数是为了在安装后自动将该依赖加入到`package.json`中。然后我们改造一下`bin/autogo.js`。
```js
#! /usr/bin/env node
var gs = require('../lib/generateStructure');

gs("demo");
```

然后执行
```bash
autogo
```

可以看到当前目录下生成了一个`demo`文件夹，里面包含了和`structure`相同的文件结构。

我们的目标已经初步达成了，接下来我们就来细化该命令。

## 命令参数

上面的命令中，我们执行`autogo`时，是生成了一个固定的`demo`项目，实际上这个名字是不能写死的，而是应该通过命令中的参数传进去。像下面这样：
```bash
autogo demo
```

因此，我们得在`bin/autogo.js`中去接收参数。为了方便起见，我们这里直接使用一个专门用于处理命令行工具的包`commander`（[文档](https://www.npmjs.com/package/commander)）。

同样，首先安装
```bash
$ npm install commander --save
```

然后改造`bin/autogo.js`为：
```js
#! /usr/bin/env node
var program = require('commander'),
    gs = require('../lib/generateStructure');

program
  .version(require('../package.json').version)
  .usage('[options] [project name]')
  .parse(process.argv);

var pname = program.args[0];

gs(pname);
```
这里的`.version()`意思是返回该命令包的版本号，即运行
```bash
autogo --version //- 返回1.0.0
```

会返回`package.json`中定义的版本号。

`.usage()`显示基本使用方法。执行

```bash
autogo --help
```

会输出：
```bash
Usage: autogo [options] [project name]

Options:
    -h, --help      output usage information
    -V, --version   output the version number
```

可以看到`Commander`帮我们做好了用法（`Usage`）信息，以及两个参数`(Options) -h, --help`和`-V， --version`。

`.parse(process.argv);`是将接收到的参数加入`Commander`的处理管道。

`program.args`是获取到命令后的参数，注意这里是一个数组。
```bash
autogo          //- 返回  []
autogo demo     //- 返回  ['demo']
autogo demo hello    //- 返回  ['demo', 'hello']
```

这里我们取第一个参数作为项目名，然后调用
```js
var pname = program.args[0];
gs(pname);
```

现在我们执行：
```bash
autogo demo2
```

就可以看到新的项目`demo2`生成了，看上去我们已经完成工作了，只要运行`autogo <项目名>`就可以生成一个新的项目结构，里面包含了处理`Sass`、`Coffee`、`jage`的`gulp`构建工具。

如果我们直接运行`autogo`是会报错的，因为没有传入项目名，实际上我们在运行一个命令而不传入任何参数时，可以直接返回帮助信息：
```js
var pname = program.args[0];
if (!pname) program.help();
```

上面我们判断是否存在参数，如果不存在就调用`program.help()`方法，这是`commander`为我们提供的显示帮助信息的方法，可以直接调用。

那有同学要说了，我不想用`jade`，就喜欢写原生`HTML`，很明显我们做了多余的事，而且整个结构就不那么合理了，我们需要的是一个干净的项目结构。

这个时候我们需要把与`jade`相关的文件删除掉（这里不是删`structure`目录下的文件，而是新项目下的指定文件）。与`jade`有关的文件有：
`/structure/views/`下的`index.jade`和`layouts/layout.jade`

* `/structure/gulpfile.js`中的`templates`任务代码

因此，我们得把上面这些文件和代码干掉。

### 移除指定模块

首先，我们创建一个`lib/jadeWithout.js`用来移除`jade`：
```js
var Promise = require('bluebird'),
    fs = Promise.promisifyAll(require('fs-extra')),
    del = require('../lib/delFile');

var files = ['/views/layouts/layout.jade', '/views/index.jade'];
function jadeWithout(project) {
  return Promise.all([del(project, files)])
    .then(function() {
      return console.log('remove jade success');
    });
}
module.exports = jadeWithout;
```

这里我们将指定的files数组中的文件都删除了，这里我们用了一个公共的删除文件模块`/lib/delFile.js`：
```js
var Promise = require('bluebird'),
    fs = Promise.promisifyAll(require('fs-extra'));

function del(project, files) {
  return files.map(function(item) {
    return fs.removeAsync(project + item);
  });
}
function delFile(project, files) {
  return Promise.all(del[project, files]);
}
module.exports = delFile;
```
因为我们这里不光有`jade`，还有`sass`和`coffee`可以被移除，所以我们创建一个公共入口`withoutFile.js`：
```js
var Promise = require('bluebird');

function deal(project, outs) {
  return outs.map(function(item) {
    var action = require('../lib/' + item + 'Without');
    return action(project);
  });
}

function withoutFile(project, outs) {
  return Promise.all([deal(project, outs)]);
}
module.exports = withoutFile;
```

这里我们需要传入一个要移除的列表(如`['sass', 'jade']`)，然后对每个模块进行删除。
最后，我们将`withoutFile`引入到`bin/autogo.js`中：
```js
var gs = reuqire('../lib/generateStructure'),
    wf = require('../lib/withoutFile');

Promise.all([gs(pname)])
    .then(function() {
      return wf(pname, ['jade', 'sass']);
    });
```

然后我们再次执行
```bash
autogo demo
```
可看到控制台依次输出了
```bash
generate project success
remove jade success
remove sass success
```

而且目标项目中相关文件已经被删除了。

这里我们是`wf(pname, ['jade', 'sass'])`写死了`outs`参数作为测试，实际上是要再传入一个数组，那么这个数组从哪里来呢？很明显，得从命令行参数中获取。

我们希望的是这样：
```bash
autogo --without jade demo
```

### option

commander为我们提供了一个option管道来配置命令参数，修改`bin/autogo.js`:
```js
program
    .version(require('../package.json').version)
    .usage('[options] [project name]')
    .option('-W, --without <str | array>', 'generate project without some models(value can be `sass`, `coffee`, `jade`)')
    .parse(process.argv);
```

这里我们添加了`option`，其格式为`.option('-<大写标识>, --<小写全称> <可取参数类型>', '功能描述')`。

接着处理`without`参数：
```js
var outs = program.without ? [program.without] : [];

Promise.all([gs(pname)])
    .then(function() {
      return wf(pname, outs);
    });
```

然后我们再运行
```bash
autogo --without jade demo
```

可以看到这里只移除了jade模块，那如果我想移除多个呢？是不是可以这样：
```bash
autogo --without [jade, sass] demo
```

注意，这样是会报错的，因为获取到的`program.without`是一个字符串`[jade, sass]`而不是数组，所以我们可以这样：
```bash
autogo --without jade,sass demo
```

`program.without`则为`jade,sass` 然后再
```js
program.without.split(',')
```

既可以获取到一个数组了，因此我们的代码就变成了：
```js
var outs = program.without ? program.without.split(',') : [];

Promise.all([gs(pname)])
    .then(function() {
      return wf(pname, outs);
    });
```

这下我们就可以这样运行了：
```bash
autogo demo --without sass,jade
```

## 发布

到目前为止，我们开发的autogo还是在本地的，现在就该将其发布到[npm](https://www.npmjs.com/)上。

1.首先我们得[注册一个账号](https://www.npmjs.com/signup)。

2.回到项目中，执行
```bash
npm login
```

输入用户名、密码和邮箱便可将本地机器与npm连接起来了。

3.执行
```bash
npm publish
```

然后回到你的npm个人主页，就可以看到我们发布成功了。
[https://www.npmjs.com/package/autogo](https://www.npmjs.com/package/autogo)

从包的路径规则来看，是没有包含用户名的，由此可知，同名的包是不会被允许的，所以大家在跟着做的时候要给项目取一个不同的名字。

然后我们来测试一下刚刚发布的包

首先删除本地开发做的autogo链接
```bash
sudo npm unlink
```

然后
```bash
npm install autogo -g
```

注意这里需要带上`-g`参数，因为命令行是应该安装在全局环境中。安装成功后，我们切换到另外一个目录下，执行：
```bash
autogo demo
```

然后结果并非我们想象的那样：
```bash
Unhandled rejection Error: ENOENT, lstat 'structure'
    at Error (native)
```
意思是找不到`structure`，这是怎么回事呢？

实际上当我们执行`npm install autogo -g`的时候，实际上是将命令包安装在了`/usr/local/lib/node_modules/autogo`下面，所以在执行命令的目录下是找不到`structure`文件夹的。

那该怎么办呢？我们能想到的就是，得在程序中去获取这个包安装的实际路径。

幸运的是Node给我们提供了`__dirname`这个变量用于获取当前执行文件的路径。我们在`lib/generateStructure.js`下`console.log(__dirname)`会输出`/usr/local/lib/node_modules/autogo/lib`，然后我们把后面的`lib`去掉就是根目录了：
```js
var root = __dirname.replace(/autogo\/lib/, 'autogo/');

function generateStructure(project, outs) {
  return fs.copyAsync(root + 'structure', project)
    .then(function(err) {
      return err ? console.error(err) : console.log('generate project');
    });
}
```

修改后，我们按照下面的方式更新，重新安装，然后
```bash
autogo demo
cd demo
npm install
gulp watch
```

OK，一个新的项目诞生了，准备开发吧...

## 更新
首先修改`package.json`配置文件中的`version`字段，比如这里我从`0.1.0`改成`0.1.1`（只能大于当前版本），然后再次
```bash
npm publish
```

即可成功发布新版本。

想将该项目从`npm`中移除吗？执行：
```bash
npm unpublish autogo --force
```
附：[项目代码](https://github.com/bimohxh/autogo)

> 原文出处：https://www.awesomes.cn/source/12
> 代码源自：[autogo.js](https://github.com/bimohxh/autogo/blob/master/bin/autogo.js#L1)

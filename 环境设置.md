# 环境设置

## 安装

我们建议使用 Python 内置的虚拟环境, 但是你也可以直接安装 Cairo. 如果要创建并进入虚拟环境, 输入:

```python
python3.9 -m venv ~/cairo_venv
source ~/cairo_venv/bin/activate
```

当 venv 被激活时 - 你应该可以在命令行中看到(cairo_venv)字样.

确保你可以安装以下 pip 包: `ecdsa`, `fastecdsa`, `sympy` ( 如果还未安装, 可以通过以下命令安装: `pip3 install ecdsa fastecdsa sympy` ). 此外, 要保证上述 pip 包可以顺利安装, 在 Ubuntu 中, 你还应该先运行:

```shell
sudo apt install -y libgmp3-dev
```

在 Mac 中, 你可以使用 `brew` 安装:

```shell
brew install gmp
```

> 注: 如果 Mac 在安装 gmp 之后依旧无法安装上述包 ( ecdsa fastecdsa sympy ), 可以尝试使用以下命令:
>
> `CFLAGS=-I/opt/homebrew/opt/gmp/include LDFLAGS=-L/opt/homebrew/opt/gmp/lib pip install ecdsa fastecdsa sympy`

接下来, 输入命令, 安装 `cairo-lang` :

```python
pip3 install cairo-lang
```



或者, 你也可以选择将包 (`cairo-lang-0.10.1.zip`) 由 [https://github.com/starkware-libs/cairo-lang/releases/tag/v0.10.1](https://github.com/starkware-libs/cairo-lang/releases/tag/v0.10.1) 下载到本地后, 运行指令安装: 

```
pip3 install cairo-lang-0.10.1.zip
```

经由测试 Cairo 可以在 ` python3.9` 中运行. 为了让他在 `python3.6` 下也能顺利运行, 你还应该安装 `contextvars`: 

```python
pip3 install contextvars
```

## 编译并运行一个 Cairo 程序

1. 创建一个名为 `test.cairo` 的文件, 输入以下内容: 

   ```
   func main() {
       [ap] = 1000, ap++;
       [ap] = 2000, ap++;
       [ap] = [ap - 2] + [ap - 1], ap++;
       ret;
   }
   ```

2. 编译( 请确保所有的命令都是在虚拟环境下执行 ): 

   ```
   cairo-compile test.cairo --output test_compiled.json
   ```

3. 运行程序:

   ```
   cairo-run \
     --program=test_compiled.json --print_output \
     --print_info --relocate_prints
   ```

4. 你还可以通过 `cairo-run` 的 `--tracer` 参数, 使用 `Cairo tracer`, 然后通过以下地址查看: [http://localhost:8100/](http://localhost:8100/) .

## Visual Studio Code 设置

从 [https://github.com/starkware-libs/cairo-lang/releases/tag/v0.10.1](https://github.com/starkware-libs/cairo-lang/releases/tag/v0.10.1) 下载Visual Studio Code扩展, 然后用以下指令安装: 

```
code --install-extension cairo-0.10.1.vsix
```

配置 Visual Studio Code :

```
"editor.formatOnSave": true,
"editor.formatOnSaveTimeout": 1500
```

**注意**: 你应该在运行虚拟环境的终端中通过指令启动 Visual Studio Code。如果你使用的是 macOS ，请参阅[此处](https://code.visualstudio.com/docs/setup/mac#_launching-from-the-command-line)。






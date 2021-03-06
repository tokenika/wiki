If you are a Windows C++ developer and MS Visual Studio is your favorite IDE, you might wonder if it's possible to have the EOS code compiled directly on Windows.

We have made several attempts with Windows C++ compilers, but to no avail. The main problem is that, while the EOS code is branched with the `WIN32` flag, unfortunately it's been done inconsistently. Also, Windows *Clang* compiler behaves differently than its Unix counterpart - it seems to be more restrictive. For example expressions like `std::move(u8"env")` are invalid in Windows.

However, we've come up with an alternative solution: [Windows Subsystem for Linux](https://msdn.microsoft.com/en-us/commandline/wsl/about) combined with [Visual Studio Code](https://code.visualstudio.com/) IDE. As you'll see this approach works even better than having the code compiled directly in Windows.

# Prerequisites

You need to be running Windows build 16215 or later. To verify your Windows version open the *Settings Panel* and then navigate to `System > About`.

*Windows Subsystem for Linux* can be only installed on the system drive. If you're running low on your disk space (extra 3-4 GB will be needed), you might want to expand your system partition using tools available [here](https://www.partition-tool.com/).

# Tooling up

First we will enable *Windows Subsystem for Linux* and then we will access the Bash command line interface from within *Visual Studio*. 

### Windows Subsystem for Linux

[Get Ubuntu on Windows](#www.microsoft.com/en-us/store/p/ubuntu/9nblggh4msv6?rtc=1) (from the Windows Store) free, clicking the 'Get the app' button there. In a result, you get an Ubuntu terminal open. Follow instructions.

Consider enabling Windows Select-Copy-Paste shortcuts on the bash console: `Defaults > Console Windows Properties > Quick Edit Mode +`

Finally update & upgrade Ubuntu:
```
sudo apt update
sudo apt full-upgrade -y 
sudo apt install -y build-essential
```
### Visual Studio Code

Download and install *Visual Studio Code* from [the official website](https://code.visualstudio.com/Download). To enable Ubuntu bash console inside *Visual Studio Code* you need to modify the *User Settings*:
   -- Navigate to `File > Preferences > User Settings`.
   -- Add the following entry in the right-hand side panel to overwrite the default settings: `"terminal.integrated.shell.windows": "C:\\Windows\\sysnative\\bash.exe"`.
   -- After saving the changes, you should be able to toggle the Ubuntu console with `Ctrl + '` or `View > Integrated Terminal`.

Optionally, consider adding C++ extensions to *Visual Studio Code*, as they can be useful for C++ development. Use `Ctrl + Shift + X` to open the *Extensions Panel* and then you might want to add the following extensions:
   -- C/C++
   -- C++ Intelisense
   -- CMake
   -- CMakeTools
   -- CMake Tools Helper
   -- Code Runner

# Compile the Source Code

Create a workspace location for EOS on the Windows file system. In this guide we will be using `X:\Workspaces\EOS` but obviously it's up to you to choose your own location.

On the Linux file system, the above location will be mapped as `/mnt/x/Workspaces/EOS`. The location you have chosen will be mapped accordingly. 

NOTE: use lower case for the name of the drive, in our case it's `/mnt/x/`.

At this stage you can start using the Ubuntu shell available in *Visual Studio Code* (`View > Integrated Terminal`). The main advantage is its support for an easy way to *copy-paste* commands. 

All the following commands are to be run in an Ubuntu shell.

1. Define the following system variable:
```
export EOSIO_INSTALL_DIR=/mnt/x/Workspaces/EOS/eos
```
NOTE: make sure to replace `x/Workspaces/EOS` with the appropriate path that matches the workspace location you have chosen on your computer.

2. Save the above system variable to the `~/.bashrc` file:
```
echo "export EOSIO_INSTALL_DIR=${EOSIO_INSTALL_DIR}"  >> ~/.bashrc
```

3. Install `cmake` and `git`:
```
sudo apt install cmake
sudo apt install git
```

4. Clone the source code from the EOS repository:
```
cd ${EOSIO_INSTALL_DIR}/../
git clone https://github.com/eosio/eos --recursive
```
4'. You can clone the EOS repository with windows tools, as well, but be sure to set the linux line ending style. With the *Windows Command Prompt*, for example:
```
git config --global core.autocrlf false
git clone https://github.com/eosio/eos --recursive
```

5. Now you are ready to proceed with the actual compilation of the source code. This step can take several hours, depending on your computer's power, and will require you to intermittently confirm some actions and also to supply the `sudo` password.
```
cd ${EOSIO_INSTALL_DIR}
./build.sh ubuntu full
```
NOTE: As this process requires downloading a lot of files from various sources, it might fail. In this case just try running this step again.

# Run the Executables

If no errors have occurred, you should have the EOS code on your Windows system compiled, resulting with multiple libraries and executables. Executables are placed in the `${EOSIO_INSTALL_DIR}/build/programs` folder:

- `eosd` - a server-side blockchain node component,
- `eosc` - a command line interface to interact with the blockchain,
- `eos-walletd` - an EOS wallet,
- `launcher` - an application for network composing and deployment, more information is available [here](https://github.com/EOSIO/eos/blob/master/testnet.md).

At this stage you should be able to run tests described in `eos/README.md`. 

For completeness, let's try if EOS can be started:
```
cd ${EOSIO_INSTALL_DIR}/build/programs/eosd && ./eosd
```
At this stage it should exit with an error complaining about the `genesis.json` file not being defined. In case it does not exit with this error, you can always close it with `ctrl + C`.

# Produce Blocks

Figure out the path for your `genesis.json` file. In our case it's `/mnt/x/Workspaces/EOS/eos/build/genesis.json`, yours will probably be different, depending on the workspace location you have chosen.

Edit the `config.ini` file (in our case it's located here: `X:\Workspaces\EOS\eos\build\programs\eosd\data-dir\config.ini`) using *WordPad* (or other text editor of your choice), locate the `enable-stale-production` entry and make it `enable-stale-production = true`.

Then append the following content in `config.ini`:
```
genesis-json = /mnt/x/Workspaces/EOS/eos/build/genesis.json
producer-name = inita
producer-name = initb
producer-name = initc
producer-name = initd
producer-name = inite
producer-name = initf
producer-name = initg
producer-name = inith
producer-name = initi
producer-name = initj
producer-name = initk
producer-name = initl
producer-name = initm
producer-name = initn
producer-name = inito
producer-name = initp
producer-name = initq
producer-name = initr
producer-name = inits
producer-name = initt
producer-name = initu

plugin = eosio::producer_plugin
plugin = eosio::wallet_api_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin 
```
NOTE: Make sure to set the proper value for the `genesis.json` path - most probably your path will be different than the one quoted above.

At this stage, when you run `eosd` again, it should start block production:
```
cd ${EOSIO_INSTALL_DIR}/build/programs/eosd && ./eosd
```

This is what you should see in your console if everything works OK:
```
232901ms thread-0   chain_plugin.cpp:80           plugin_initialize    ] initializing chain plugin
232902ms thread-0   producer_plugin.cpp:159       plugin_initialize    ] Public Key: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
232903ms thread-0   http_plugin.cpp:132           plugin_initialize    ] host: 127.0.0.1 port: 8888
232905ms thread-0   http_plugin.cpp:135           plugin_initialize    ] configured http to listen on 127.0.0.1:8888
...
232998ms thread-0   producer_plugin.cpp:170       plugin_startup       ] producer plugin:  plugin_startup() begin
232999ms thread-0   producer_plugin.cpp:175       plugin_startup       ] Launching block production for 19 producers.

*******************************
*                             *
*   ------ NEW CHAIN ------   *
*   -   Welcome to EOS!   -   *
*   -----------------------   *
*                             *
*******************************

Your genesis seems to have an old timestamp
Please consider using the --genesis-timestamp option to give your genesis a recent timestamp

233012ms thread-0   producer_plugin.cpp:185       plugin_startup       ] producer plugin:  plugin_startup() end
233012ms thread-0   http_plugin.cpp:147           plugin_startup       ] start processing http thread
...
233059ms thread-0   http_plugin.cpp:224           add_handler          ] add api url: /v1/account_history/get_transaction
233060ms thread-0   http_plugin.cpp:224           add_handler          ] add api url: /v1/account_history/get_transactions
237003ms thread-0   chain_controller.cpp:235      _push_block          ] initq #1 @2017-09-29T16:03:57  | 0 trx, 0 pending, exectime_ms=0
237005ms thread-0   producer_plugin.cpp:233       block_production_loo ] initq generated block #1 @ 2017-09-29T16:03:57 with 0 trxs  0 pending
240003ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-29T16:04:00  | 0 trx, 0 pending, exectime_ms=0
240004ms thread-0   producer_plugin.cpp:233       block_production_loo ] initc generated block #2 @ 2017-09-29T16:04:00 with 0 trxs  0 pending
243003ms thread-0   chain_controller.cpp:235      _push_block          ] initd #3 @2017-09-29T16:04:03  | 0 trx, 0 pending, exectime_ms=0
```
NOTE: It might happen that `eosd` hangs and fails to produce blocks at the first attempt. In this case just exit the process using `ctrl + C`, then wait a bit and try again.

# Update the Source Code

In order to update the source code from the official repository and recompile it, run the following commands:
```
cd ${EOSIO_INSTALL_DIR}
git reset --hard # if you want to revert all local changes
git pull
rm -r build && mkdir build && cd build

cmake -DCMAKE_BUILD_TYPE=Debug ../ 
make
```

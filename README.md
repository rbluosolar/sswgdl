# Run SSW with GDL

Our tests are very limited, it is not guaranteed that all SSW programs work well with GDL. And we only tested our way in Linux and macOS, while this way should also work well in Windows.

Welcome for reporting issues to [Mr. Luo, RunBin (罗润彬)](mailto:rbluo@mail.ustc.edu.cn) or [Dr. Chen, Jun (陈俊)](mailto:chenjun@pmo.ac.cn)

-----------------------------
## Software requirements
* tcsh: https://www.tcsh.org/
* SSW: http://www.lmsal.com/solarsoft/
  * `$SSW` in the following file `sswgdl` is the SSW top level path, e.g. `/usr/local/ssw`,`$HOME/ssw`, `C:\ssw`
* GDL: https://gnudatalanguage.github.io/downloads.html, 
  `$GDL_DIR` in the following file `sswgdl` is the GDL top level path
  * If you install GDL from https://github.com/gnudatalanguage/gdl/releases, `$GDL_DIR` is where you `unzip` the package
  * If you install GDL from the software source of the system, `$GDL_DIR` could be:
    * Ubuntu & Fedora:  /usr/share/gnudatalanguage
    * ArchLinux: /usr/lib/gdl
    * Gentoo: /usr/local/share/gdl
    * macOS: /opt/local/share/gnudatalanguage

## Make GDL “pretend” to be IDL
* If you install GDL from https://github.com/gnudatalanguage/gdl/releases
  ```
  ln -s $GDL_DIR/bin/gdl $GDL_DIR/bin/idl
  ```
* If you install GDL from the software source of the system, please make a directory `bin/` under `$GDL_DIR`
  ```
  mkdir $GDL_DIR/bin/
  ```
  please locate the position of `$command_gdl` in the system
  ```
  whereis gdl
  ```
  It displays `/opt/local/bin/gdl` in macOS, it displays `/usr/bin/gdl` in Ubuntu. Then execute
  ```
  ln -s $command_gdl $GDL_DIR/bin/idl
  ```

## Launch SSW with a script

* Create a file `sswgdl` (can be any other name) with the following content:

  ```tcsh
  #!/bin/tcsh

  setenv SSW $SSW

  # Point IDL_DIR to the GDL top level path
  setenv IDL_DIR $GDL_DIR

  # Set GDL_PATH that including $SSW
  setenv GDL_PATH $IDL_DIR/lib:+$SSW

  setenv SSW_INSTR "gen sdo aia hmi xrt hessi ontology"
  source $SSW/gen/setup/setup.ssw

  # Launch SSW with GDL
  sswidl
  ```
* Make the script executable
  ```tcsh
  chmod +x sswgdl
  ```
* launch SSW with GDL in a terminal:
  ```
  ./sswgdl
  ```
  * note: put `sswgdl` to the `$path` of the system is suggested, e.g. you can append this line to `~/.bashrc`
    ```
    export PATH=$HOME/bin:$PATH
    ```
    then `$HOME/bin` is a `$path` of the system. If `sswgdl` is putted in a `$path`, you can directly run SSW with GDL
    ```
    sswgdl
    ``` 
* Confirm the path inside the session
  ```idl
  GDL> PRINT, !PATH
  ```

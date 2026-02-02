# Run SSW with GDL

The idea is to make GDL pretend to be IDL.

GDL is fully syntax-compatible with IDL 7.1 and partially compatible with IDL 8.x. Due to the limited scope of our tests, compatibility with all SSW programs is not guaranteed. And we only tested our way in Linux and macOS, while this way should also work well in Windows.

Please address comments and suggestions to [Mr. Luo, RunBin (罗润彬)](mailto:rbluo@mail.ustc.edu.cn) or [Dr. Chen, Jun (陈俊)](mailto:chenjun@pmo.ac.cn)

-----------------------------
## Software requirements
* tcsh: https://www.tcsh.org/
* SSW: http://www.lmsal.com/solarsoft/
  * `$SSW` in the following file `sswgdl` is the SSW top level path, e.g. `/usr/local/ssw`,`$HOME/ssw`, `C:\ssw`
* GDL: https://gnudatalanguage.github.io/downloads.html
  * `$GDL_DIR` in the following file `sswgdl` is the GDL top level path
  * if you install GDL from the software source of the system
    * command of install:
      * Ubuntu/Debian: `apt install gnudatalanguage`
      * Fedora: `yum install gdl`
      * ArchLinux: `pacman -S gnudatalanguage`
      * Gento: `emerge gdl`
      * macOS:`port install gnudatalanguage` or `fink install gdl`
    * then `$GDL_DIR` should be:
      * Ubuntu & Fedora: `/usr/share/gnudatalanguage`
      * ArchLinux: `/usr/lib/gdl`
      * Gentoo: `/usr/local/share/gdl`
      * macOS: `/opt/local/share/gnudatalanguage`
    * please locate the position of $command_gdl in the system
      ```
      whereis gdl
      ```
      It would display `/opt/local/bin/gdl` in macOS, it would display `/usr/bin/gdl` in Ubuntu. Please execute the following command to make GDL pretend to be IDL:
      ```
      mkdir $GDL_DIR/bin/
      ln -s $command_gdl $GDL_DIR/bin/idl
      ```
    * `$GDL_LIB` in the following file `sswgdl` is `$GDL_DIR/lib`, `lib.7z` is unnecessary in this case
  * If you download GDL from https://github.com/gnudatalanguage/gdl/releases
    * `$GDL_DIR` is where you `unzip` the package, it should contain `bin/gdl`
    * please execute the following command to make GDL pretend to be IDL.
      ```
      ln -s $GDL_DIR/bin/gdl $GDL_DIR/bin/idl
      ```
    * `$GDL_LIB` in the following file `sswgdl` is the full path of `lib/` unpack from `lib.7z`
      ```
      7z x lib.7z o lib
      ```
  * If you compile the project from https://github.com/gnudatalanguage/gdl 
    * `$GDL_DIR` is where you build the project, it should contain `src/gdl`

    * please execute the following command to make GDL pretend to be IDL.
      ```
      mkdir $GDL_DIR/bin/
      ln -s $GDL_DIR/src/gdl $GDL_DIR/bin/idl
      ```
    * `$GDL_LIB` in the following file `sswgdl` is the full path of `lib/` unpack from `lib.7z`
      ```
      7z x lib.7z o lib
      ```
  * **Potential issue**: If launching SSW with GDL 1.0.0, `% Cannot apply operation to datatype STRING` will be printed. It is unclear which specific programs of SSW are affected, but this issue will not happen with GDL 1.1.3

## Launch SSW with a script

* Create a file `sswgdl` (can be any other name) with the following content:

  ```tcsh
  #!/bin/tcsh

  # please replace the text `$SSW` to the SSW top level path
  setenv SSW $SSW

  # Point IDL_DIR to the GDL top level path
  # please replace the text `$GDL_DIR` the GDL top level path
  setenv IDL_DIR $GDL_DIR

  # please replace the text `$GDL_LIB` by the directory of `lib/`
  # setenv GDL_LIB $GDL_LIB

  # Set GDL_PATH that including $SSW and your private path for *.pro
  # the text `$SSW` in this line is unnessary to change
  setenv GDL_PATH $GDL_LIB/:+$SSW/:+$HOME/your/private/pro/path

  # set your instruments
  setenv SSW_INSTR "gen sdo aia hmi xrt hessi ontology"

  # the text `$SSW` in this line is unnessary to change
  source $SSW/gen/setup/setup.ssw

  # Launch SSW
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
    then `$HOME/bin` is a `$path` of the system.
    If `sswgdl` is putted in a `$path`, you can directly run this command at any directory
    ```
    sswgdl
    ``` 
* Confirm the path inside the session
  ```idl
  GDL> PRINT, !PATH
  ```

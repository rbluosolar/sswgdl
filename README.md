# Run SSW with GDL (macOS + tcsh)

## Goal

Make **SSW** think it is launching **IDL**, while it actually runs **GDL**.

This setup:

- provides an `idl` executable where SSW expects it,
- prepends GDL’s built-in library path to `IDL_PATH`,
- keeps SSW’s normal startup workflow.

> No system-wide symlink (e.g. `/opt/local/bin/idl`) is used.

## Requirements

- macOS
- `tcsh`
- SSW installed locally
- GDL installed locally

## Placeholders (replace with your own paths)

- `<SSW_ROOT>`: the root directory of your SSW installation
- `<GDL_PREFIX>`: the installation prefix of your GDL (contains `bin/gdl`)

Expected:

- GDL executable: `<GDL_PREFIX>/bin/gdl`
- GDL libraries (one of these usually exists; use the one that matches your install):
  - `<GDL_PREFIX>/share/gnudatalanguage/lib/`
  - `<GDL_PREFIX>/gnudatalanguage/lib/`

## Steps (1–5)

### 1) Make GDL “pretend” to be `idl` (local symlink only)

SSW typically runs IDL via `$IDL_DIR/bin/idl`. Create a local symlink inside your GDL prefix:

```tcsh
ln -s <GDL_PREFIX>/bin/gdl <GDL_PREFIX>/bin/idl
```

### 2) Copy `ssw_idl` to `ssw_gdl`

In `<SSW_ROOT>/gen/setup/`:

```tcsh
cp <SSW_ROOT>/gen/setup/ssw_idl <SSW_ROOT>/gen/setup/ssw_gdl
chmod +x <SSW_ROOT>/gen/setup/ssw_gdl
```

### 3) Modify `ssw_gdl` to prepend GDL libraries to `IDL_PATH`

Edit `<SSW_ROOT>/gen/setup/ssw_gdl`. Locate the section that sets `IDL_PATH`, then set:

```tcsh
# set the darn thing.
setenv GDL_LIB "+<GDL_PREFIX>/share/gnudatalanguage/lib/"

setenv IDL_PATH "$GDL_LIB":"$TEMP2"
alias idltool $IDL_DIR/bin/idltool

# call IDL with any arguments (actually GDL via the idl symlink)
onintr -                                # allow ctrl c in idl
$IDL_DIR/bin/idl $command               # no alias (see comment) slf 30-Jul

cleanup:
onintr -
```

### 4) Add an `sswgdl` alias in `setup.ssw`

Edit `<SSW_ROOT>/gen/setup/setup.ssw`. In the “Define idl command for SSW” block, add:

```tcsh
alias sswgdl $SSW/gen/setup/ssw_gdl
```

Example result:

```tcsh
###################### Define idl command for SSW ###########################
alias sswidl $SSW/gen/setup/ssw_idl
alias sswgdl $SSW/gen/setup/ssw_gdl
alias sswidlde $SSW/gen/setup/ssw_idlde
if ($qdisp) echo ""
if ($qdisp) echo "Type <sswidl> to start SSW IDL"
##############################################################################
```

### 5) Create a launcher script `ssw.csh`

Create `ssw.csh`:

```tcsh
#!/bin/tcsh

setenv SSW <SSW_ROOT>
setenv SSW_INSTR "gen sdo aia hmi xrt hessi ontology"

# Point IDL_DIR to the GDL installation prefix
setenv IDL_DIR "<GDL_PREFIX>"

source $SSW/gen/setup/setup.ssw

# Launch SSW with GDL
sswgdl
```

Make it executable and run:

```tcsh
chmod +x ssw.csh
./ssw.csh
```

## Quick checks

### Confirm `idl` points to `gdl`

```tcsh
ls -l <GDL_PREFIX>/bin/idl
```

### Confirm the path inside the session

At the GDL/IDL prompt:

```idl
PRINT, !PATH
```

You should see your GDL lib path at the beginning, followed by SSW paths.

## Notes / Limitations

GDL is not 100% IDL-compatible. Some IDL-specific or licensed components may not work under GDL.

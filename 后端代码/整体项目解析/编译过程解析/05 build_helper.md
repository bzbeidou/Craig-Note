### `install_package`

系统上对应软件的脚本代码，

```bash
function install_package()
{
    which dpkg &> /dev/null || return  
    local name=$1 
    DEBIAN_FRONTEND=noninteractive
    dpkg -l $name | grep ii\ \  || {
        echo "Install $name"
        $sudo apt update
        $sudo apt install -y $name
    }
}
```


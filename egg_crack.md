1、安装完centos，开启转发
# nano /etc/sysctl.conf
新增一行：net.ipv4.ip_forward = 1
生效：
# sysctl -p

2、安装oas
# yum -y install https://as-repository.openvpn.net/as-repo-centos7.rpm
# yum -y install openvpn-as
# passwd openvpn
或离线安装
# yum -y install openvpn-as-bundled-clients-25.rpm
# yum -y install openvpn-as-2.11.0_794ab41d-CentOS7.x86_64.rpm
（安装DCO：apt install -y openvpn-dco-dkms）（重启后要输入密码验证才会启用）

3、crack
# yum install -y zip unzip epel-release vim curl wget net-tools
（Ubuntu：sudo apt install python3-pip）（需解决uncompyle6 3.10的兼容问题）
(修改兼容3.10.12：/usr/local/lib/python3.10/dist-packages/xdis/magics.py，add_canonic_versions增加3.10.12)
(或者用更好用的工具反编译pycdc：https://blog.csdn.net/qq_63585949/article/details/127080253)
(在线反编译：https://tool.lu/pyc/)
# systemctl stop openvpnas
# cd /usr/local/openvpn_as/lib/python
# mkdir unlock_license
# mv pyovpn-2.0-py3.6.egg pyovpn-2.0-py3.6.egg_bak
# cp -rp pyovpn-2.0-py3.6.egg_bak unlock_license/pyovpn-2.0-py3.6.zip
# cd unlock_license/
# unzip pyovpn-2.0-py3.6.zip
# cd pyovpn/lic/
# pip3 install uncompyle6
# uncompyle6 uprop.pyc > uprop.py
# nano uprop.py
新增一行：ret['concurrent_connections'] = 2048
位置在：
        apc = self._apc()
            v_agg += apc
            if ret == None:
                ret = {}
            ret[prop] = max(v_agg + v_nonagg, bool('v_agg') + bool('v_nonagg'))
            ret['apc'] = bool(apc)
            if DEBUG:
                print("ret['%s'] = v_agg(%d) + v_nonagg(%d)" % (prop, v_agg, v_nonagg))
        ret['concurrent_connections'] = 2048
        return ret
# rm -f uprop.pyc
# python3 -O -m compileall uprop.py && mv __pycache__/uprop.cpython-36.opt-1.pyc uprop.pyc
# rm -rf __pycache__ uprop.py
# cd /usr/local/openvpn_as/lib/python/unlock_license
# zip -r pyovpn-2.0-py3.6_cracked.zip common EGG-INFO pyovpn
# mv pyovpn-2.0-py3.6_cracked.zip pyovpn-2.0-py3.6.egg
# mv pyovpn-2.0-py3.6.egg ../ && cd ..
# cd /usr/local/openvpn_as/lib/python
# rm -rf __pycache__/*
# cd /usr/local/openvpn_as/bin
# ./ovpn-init

DELETE
yes





2.9.4 及以上是 /pyovpn/lic/uprop.pyc; 按照网上流行的破解方法，把这个文件解压出来并改名为uprop2.pyc, 然后新建一个 uprop.py 文件
内容：
from pyovpn.lic import uprop2
old_figure = None

def new_figure(self, licdict):
    ret = old_figure(self, licdict)
    ret['concurrent_connections'] = 1024
    return ret


for x in dir(uprop2):
    if x[:2] == '__':
        continue
    if x == 'UsageProperties':
        exec('old_figure = uprop2.UsageProperties.figure')
        exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))

再将上面的 uprop.py 编译为库文件uprop.pyc
注意 uprop.cpython-37.opt-1.pyc 文件名会随着 python 版本变化而变化.

some notes here.

compile option for GL-MIFI 2016 with 4G
```
make image PROFILE=DOMINO PACKAGES="luci uhttpd openvpn-openssl luci-app-openvpn openvpn-easy-rsa comgt luci-proto-3g usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi kmod-usb-serial-option kmod-usb-serial kmod-usb-serial-wwan"
```

compile for GL-MT300N
```
make image PROFILE=GL-MT300N PACKAGES="luci uhttpd openvpn-openssl luci-app-openvpn openvpn-easy-rsa"
```

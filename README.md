# DHCP-CF
Termux Network DHCP Tool  An IPv4 DHCP/DNS toolkit

pkg update
pkg install unzip curl nano
termux-setup-storage

unzip "$HOME/storage/downloads/termux-netdhcp.zip" -d "$HOME"
cd "$HOME/termux-netdhcp"
bash install-termux.sh
./ndhcp doctor


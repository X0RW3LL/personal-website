# Functions & Aliases

Hot tip: don't flood your `~/.bashrc` with function/alias declarations. You can offload those to `~/.bash_aliases` instead if your distro sources it by default

This is going to include some of the aliases I personally like to use, so here goes nothing

## Aliases

```sh
alias update='sudo apt update' # update the repo index
alias uplist='apt list --upgradable' # show packages pending upgrade
alias upgrade='sudo apt full-upgrade -y' # full-upgrade, yes
alias _cleanup='sudo apt autoclean && sudo apt autoremove'
alias _wg='sudo wg-quick up wg0'
alias _diswg='sudo wg-quick down wg0'
alias ipa='ip -br a'
alias ipr='ip r'
alias _jctl='sudo journalctl --vacuum-time=1d' # clean up the journal
alias xc="xsel -b -i < $1" # Depends: xsel - copy the contents of file ($1) into clipboard: xc /path/to/file
alias cnf="command-not-found --ignore-installed $1" # for when you want a quick way to figure out which packages provides a certain command
alias kb='curl -L https://www.kernel.org/finger_banner' # get the latest kernel release versions
alias cpowersave='sudo cpupower frequency-set -rg powersave' # Depends: linux-cpupower
alias cschedutil='sudo cpupower frequency-set -rg schedutil'
alias cperformance='sudo cpupower frequency-set -rg performance'
alias bw='bitwise' # Depends: bitwise
```

## Functions
```sh
# For when you're using a VPN, but would like to execute some command
# using your non-VPN connection
utnl () {
	# short for un-tunnel (Home network _only_)
	if [[ $# -eq 0 ]]
	then
		echo "[!] Usage: $FUNCNAME <IFACE>"
	else
		GW=$(ip r | awk '/default/ { print $3 }')
		SUB=${GW%.*}
		sudo ip netns add home
		sudo ip netns exec home ip link set lo up
		sudo ip link add link $1 name home0 type macvlan
		sudo ip link set home0 netns home
		sudo ip -n home a add $SUB.250/24 metric 1024 brd + dev home0
		sudo ip netns exec home sysctl -q net.ipv6.conf.home0.disable_ipv6=1
		sudo ip netns exec home sysctl -qp 2>/dev/null
		sudo ip -n home link set home0 up
		sudo ip -n home r add default via $GW
	fi
}

# To be used after utnl eth0/wlan0 has already been started
nex () {
	# short for netns-exec
	y="\e[33m"
	r="\e[31m"
	m="\e[35m"
	g="\e[32m"
	e="\e[0m"
	cmd=$( printf '%q ' "${@:1}" )
	if [[ $# -eq 0 ]]
	then
		echo "[!] Usage: $FUNCNAME <command> [args]"
	elif [ $1 = "destroy" ]
	then
		if sudo ip netns delete home 2>/dev/null
		then
			printf "%b[+] home network namespace destroyed%b\n" "$g" "$e"
		else
			printf "%b[-] home network namespace not found. Skipping...%b\n" "$r" "$e"
		fi
	elif echo $1 | grep -q firefox
	then
		if ps aux | grep firefox | grep -v grep 1>/dev/null
		then
			printf "%b[-] Close already running instance(s) of Firefox and try again%b\n" "$r" "$e"
		else
			printf "[+] Executing %b|  %s |%b on %bhome%b as %b%s%b\n\n" "$y" "$cmd --display=:1" "$e" "$m" "$e" "$g" "$USER" "$e"
			sudo ip netns exec home runuser - $USER home -c "$cmd --display=:1"
		fi
	else
		printf "[+] Executing %b|  %s |%b on %bhome%b as %b%s%b\n\n" "$y" "$cmd" "$e" "$m" "$e" "$g" "$USER" "$e"
		sudo ip netns exec home runuser - $USER home -c "$cmd"
	fi
}

# Compare SHA256 checksums (useful for downloaded ISOs/packages)
_checksum () {
	g="\e[32m"
	r="\e[31m"
	e="\e[0m"
	if [[ $# -lt 2 ]]
	then
		echo "[!] Usage: $FUNCNAME <file> <SHA256>"
	else
		[ "$(sha256sum $1 | cut -d ' ' -f1)" == "$2" ] && printf "%b\n[+] SHA256 checksum OK\n\n%b" "$g" "$e" || printf "%b\n[-] SHA256 checksum mismatch\n\n%b" "$r" "$e"
	fi
}

# dmesg logs, separating message levels to different output files
dlogs () {
	dmesg -t > dmesg_current
	dmesg -t -k > dmesg_kernel
	dmesg -t -l emerg > dmesg_current_emerg
	dmesg -t -l alert > dmesg_current_alert
	dmesg -t -l crit > dmesg_current_crit
	dmesg -t -l err > dmesg_current_err
	dmesg -t -l warn > dmesg_current_warn
	dmesg -t -l info > dmesg_current_info
}
```

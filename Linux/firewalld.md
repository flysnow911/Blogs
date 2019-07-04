Firewalld是啥？
FirewallD(firewall Daemon)是iptables 的一个封装，可以让你更容易地管理iptables规则。
它并不是 iptables 的替代品。虽然 iptables 命令仍可用于 FirewallD，
但建议使用 FirewallD 时仅使用 FirewallD 命令。

常用命令：

systemctl stop firewalld
systemctl enable firewalld
systemctl disable firewalld
systemctl start firewalld
systemctl status firewalld

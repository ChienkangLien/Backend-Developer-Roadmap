# LXC
Linux Containers（LXC）是一種操作系統層面的虛擬化技術，用於在單個 Linux 系統上運行多個隔離的 Linux 環境。它提供了一種輕量級的虛擬化方式，使得可以在單個 Linux 主機上運行多個獨立的容器，每個容器都有自己的檔案系統、資源、網路等環境，彼此相互隔離。
1. 隔離性：每個容器都運行在自己的環境中，互相隔離，不會干擾或影響其他容器。
2. 效能：相比傳統的虛擬機，容器化技術通常更輕量級，因為它們共享主機核心，運行時更加高效。
3. 快速啟動：容器可以快速啟動、停止，並且應用程式可以在容器中輕鬆部署和執行。

## 主要組件
Namespace：
* PID Namespace：隔離進程ID，讓每個容器有自己的進程樹，互不干擾。
* Network Namespace：隔離網路資源，每個容器有自己的網路設定、接口和 IP 地址。
* Mount Namespace：隔離檔案系統的掛載點，使得每個容器有自己的檔案系統視圖。
* UTS Namespace：隔離主機名和域名。
* IPC Namespace：隔離進程間通信。

Cgroups（Control Groups）：
* 資源控制：控制容器可以使用的系統資源，如 CPU、記憶體、磁碟 IO、網路等。
* 限制和管理：可以設定資源限制、優先順序和配額，以避免容器競爭和濫用資源。

Container Runtime：
* 容器管理工具：例如 LXC/LXD，負責建立、啟動、停止和管理容器。
* Systemd-nspawn：另一個常用的容器執行時工具，允許啟動容器而不需要額外的軟體。

Kernel Support：
* Linux Kernel 功能：Namespace 和 Cgroups 是 Linux Kernel 的功能，提供了虛擬化的支持。
* Container Aware Kernel：一些版本的 Kernel 適用於容器化，有特定的功能和改進以更好地支持容器。


這些組件共同工作，創建出隔離的容器環境，使得每個容器看起來像是運行在獨立的環境中，但實際上它們共享了主機的 Kernel 和資源。
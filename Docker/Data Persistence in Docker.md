# Data Persistence in Docker
默認情況下，Docker 容器內的存儲是暫時性的，這意味著容器內部的任何數據更改或修改只會在容器運行時持久存在。一旦容器停止並移除，所有相關數據都將丟失。這是因為 Docker 容器天生就是無狀態的。

這種臨時或短暫的存儲稱為“臨時容器文件系統(ephemeral container file system)”。這是 Docker 的一個重要特性，因為它能夠在不同環境中快速且一致地部署應用程序，而不必擔心容器的狀態。

為了解決這個問題並在容器生命周期中保留數據，Docker 提供了各種數據持久化的方法。

## Docker Volumes
Volume mounts（數據卷掛載）是一種將主機系統上的文件夾或文件映射到容器內部的文件夾或文件的方式。這使得數據可以在容器被移除後仍然持久存在於容器外部。此外，多個容器可以共享同一個數據卷，使得容器之間的數據共享變得簡單。是持久保存由 Docker 容器生成和使用的數據的首選方式。

建立Volume：
`docker volume create my-volume`
建立一個叫做my-volume 的volume，可以透過以下指令檢查：
`docker volume inspect my-volume`

將數據卷掛載到容器，透過 -v 或 --mount：

1. `docker run -d -v my-volume:/data your-image`


2. `docker run -d --mount source=my-volume,destination=/data your-image`

在上述兩個示例中，my-volume 是我們之前創建的卷的名稱，/data 是容器內部卷將被掛載的路徑。

要在多個容器之間共享一個卷，只需在多個容器上掛載相同的卷。以下是在運行不同鏡像的兩個容器之間共享 my-volume 的方法：

1. `docker run -d -v my-volume:/data1 image1`
2. `docker run -d -v my-volume:/data2 image2`

image1 和 image2 將有權存取儲存在 my-volume 中的相同資料。

刪除數據卷
`docker volume rm my-volume`

## Bind Mounts
相比數據卷（volumes），綁定掛載（bind mounts）的功能性有限。當你使用綁定掛載時，主機機器上的文件或目錄被掛載到容器中。該文件或目錄是通過其在主機機器上的絕對路徑進行引用的。

文件或目錄不需要已經存在於 Docker 主機上。如果尚不存在，則會根據需要建立它。綁定掛載非常高效，但主機機器的文件系統需要具有特定且可用的目錄結構。

假設你有一個主機上的目錄 /host/data，你想將它掛載到容器中的 /container/data 目錄。
`docker run -v /host/data:/container/data my-volume`
這個命令會啟動一個新容器名叫my-volume，並將主機上 /host/data 目錄掛載到容器內的 /container/data 目錄。這樣容器內的數據修改會直接反映到主機的目錄中，也可以通過主機上的目錄對容器內的數據進行操作。

## Docker tmpfs mounts
將一個臨時文件系統掛載到容器的內存中，數據存儲在內存中，但在容器終止時會丟失。

`docker run --tmpfs /container/path my-volume`
在容器內創建一個臨時的 tmpfs 掛載點，用於存儲 /container/data 路徑下的數據。
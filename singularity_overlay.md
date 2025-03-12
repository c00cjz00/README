## 程式碼
- 參考 https://pawseysc.github.io/singularity-containers/32-writable-trinity/index.html
```
# 建置200MB的用富專屬空間
export SIZE="200"
export FILE="dark_lia_limit_disk"
singularity overlay create --size $SIZE $FILE

# 掛載 --overlay (最上層的空間都是你的)
singularity shell --overlay my_overlay docker://ubuntu:18.04

# 建置專屬目錄檔案   (最上層的空間都是你的)
mkdir /mytwb_紀錄
cd /mytwb_紀錄
echo "第一筆紀錄" > /mytwb_紀錄/01.record
echo "第一筆紀錄" > /mytwb_紀錄/02.record

# 離開後重新掛載overlay後, 資料都在, 但用戶無法取得這個overlay image 
exit
```

## 解說
### **1. 建立 Overlay 文件系統**
```bash
export SIZE="200"  # 設定 overlay 檔案的大小（MB）
export FILE="my_overlay"  # 設定 overlay 檔案的名稱
singularity overlay create --size $SIZE $FILE  # 建立 overlay
```
這會建立一個大小為 **200MB** 的 `my_overlay` 檔案，這個檔案可用來擴充容器的檔案系統，允許你在裡面存儲資料。

---

### **2. 掛載 Overlay 並進入容器**
```bash
singularity shell --overlay my_overlay docker://ubuntu:18.04
```
這會：
- **啟動 Ubuntu 18.04 容器**（從 Docker Hub 下載）
- **掛載 `my_overlay` 作為 overlay filesystem**，讓你能在容器內的檔案系統中讀寫

---

### **3. 在容器內創建檔案**
```bash
Singularity> mkdir /australia  # 創建目錄
Singularity> cd /australia  # 進入目錄
Singularity> echo perth > wa  # 在 "wa" 檔案中寫入 "perth"
Singularity> echo canberra > act  # 在 "act" 檔案中寫入 "canberra"
```
這段程式碼：
- 建立一個名為 `/australia` 的目錄
- 建立 `wa` 檔案，內容是 `perth`
- 建立 `act` 檔案，內容是 `canberra`

因為使用了 `overlay`，這些變更會持久存在，即使容器關閉，檔案仍然會保留。

---

### **4. 離開容器**
```bash
Singularity> exit
```
這會結束 Singularity shell，但 `my_overlay` 內的數據仍然存在。

---

### **為何使用 Overlay？**
Singularity 容器通常是 **唯讀（read-only）**，但有時候你可能需要：
1. **持久化存儲**（即使容器關閉後仍保留檔案）
2. **避免改變基礎映像檔（image）**
3. **模擬一個具有寫入權限的檔案系統**

Overlay 提供了一個簡單的方法來擴展 Singularity 的功能，而不需修改原始映像檔。

---

### **總結**
這段程式碼使用 `singularity overlay create` 來建立一個持久化的 overlay 檔案系統，並掛載到容器內，使得 `/australia` 目錄及其內容能夠在容器結束後仍被保留。這種方式適合需要存儲數據但又不想修改基礎映像的情境。

這樣的做法在 **高效能運算（HPC）環境** 下特別有用，因為它允許使用者在共享的容器環境中管理自己的數據，而不影響其他使用者或原始映像。

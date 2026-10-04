# Linux 核心設計：54 個學習單元課表

整理日期：2026-10-05

修訂日期：2026-10-05（依先備關係與教材可操作性審查調整，加入每週自我檢核）

教材主來源：[成大 Linux 核心設計課程](https://wiki.csie.ncku.edu.tw/linux/schedule)。

適用程度：已懂基本 C，進階觀念以複習方式補足。時間配置：平日每天 30–60 分鐘，週末每天 1–2 小時，先以每週約 6 小時安排。

已有環境：Ubuntu 24.04。需要操作時啟動現有環境；第 1 週略過 Linux 安裝，核對工具與核心版本即可。

學習路線：共同主線 → 驅動深入 → 虛擬化 → 即時 Linux → Rust 核心開發 → 核心開發與貢獻。

## 使用方式

- 從第 1 週往下閱讀。這份課表有 54 個學習單元，保留「第幾週」作為索引，每單元先抓一週。
- 每週先依序開啟「必讀」連結，完成後勾選，再做「自我檢核」。每週固定 4 項：3 個核心問題＋1 個短練習；若再做原有的「選做實驗」，合計 5 項。
- 自我檢核先從每週約 6 小時中預留 30–60 分鐘，包含以 15–30 分鐘為目標的短練習。先用自己的話回答，再回查教材；可用幾句話、程式註記或流程圖作答，不要求背誦。
- 能說明問題中的機制、完成短練習並留下結果，即可勾選及往下學。卡住時記下不確定之處，回看當週指定範圍；不以自行完成完整驅動或建置整顆核心作為過關條件。
- 「閱讀範圍」是當週重點，不要求把長篇教材、整本書或全部錄影一次看完。相同連結在後續週次出現時，只看新指定的部分。
- 平日先讀教材、追範例；週末回答檢核問題、完成短練習，再依時間選做實驗。短練習多可直接閱讀原始碼或畫流程，不必每週啟動新的實驗環境。
- 一週讀不完就延長，不必追趕日曆。54 個單元表示內容順序，不保證 54 個日曆週完成；環境建置與自行開發通常需要額外時間。
- 原課程的文章與錄影共用同一教材入口；若文章內有錄影，優先看本週主題對應的部分。
- 標示「補充／官方」或「補充／範例」的連結是額外整理的文件與程式，其餘教材主要取自原課程清單。這份課表是個人自學安排，不是原課程授課順序。
- 先使用現有 Ubuntu 24.04 讀教材、建置模組；第 25 週再準備專用 QEMU 環境。第 3 週起可以累積同一個模組專題，也可以只追教材範例。
- 記下核心版本、組態與範例版本；依使用版本核對 API。連到 GitHub 的程式先固定 tag／commit，再讀該版本的 README 與原始碼。

## 路線總覽

| 單元／週次 | 階段 | 完成閱讀後應能理解 |
|---|---|---|
| 1–6 | C 複習、核心模組、程式執行與行程 | 模組生命週期，以及使用者程式進入核心的流程 |
| 7–12 | 記憶體與同步 | 理解資料配置、共享與競爭 |
| 13–19 | 裝置讀寫、事件通知、中斷與裝置模型 | 裝置範例如何讀寫、等待與回報事件 |
| 20–24 | 網路、觀測與共同專題 | 連結應用程式 I/O 與核心行為 |
| 25–30 | 驅動深入 | PCI 配對、MMIO、IRQ、DMA 與資源清理 |
| 31–36 | 虛擬化 | 理解 QEMU／KVM 與小型 VMM 的基本流程 |
| 37–42 | 即時 Linux | 排程政策、搶佔與延遲量測的意義 |
| 43–50 | Rust 核心開發 | Rust 最小模組的生命週期與資源管理 |
| 51–54 | 核心開發與貢獻 | 問題重現、修改驗證與 patch 審查流程 |

## 第 1–6 週：從複習到核心執行

### 第 1 週｜環境、指標與函式呼叫

**必讀（依序開啟）**

環境入口：直接啟動已有的 Ubuntu 24.04，核對 gcc、make、git 與 uname -r。第 3 週使用該環境正在執行的核心所對應的標頭；不預先指定核心版號，因為 Ubuntu 更新與核心套件可能不同。專用 QEMU 核心建置留到第 25 週，現有 Ubuntu 可作為 Linux 建置端。

- [ ] 1. [開發工具整理](https://hackmd.io/@sysprog/gnu-linux-dev/)：核對現有 Ubuntu 的編譯工具、Git 與 make；Linux 安裝與熟悉的操作略過。
- [ ] 2. [C 指標](https://hackmd.io/@sysprog/c-pointer)：複習陣列與指標、結構、生命週期、函式指標。
- [ ] 3. [函式呼叫](https://hackmd.io/@sysprog/c-function)：先看呼叫流程與參數傳遞；組合語言細節可後補。

**自我檢核（4 項）**

- [ ] 1. 陣列與指標有哪些差別？陣列作為函式參數時，為什麼不能直接用 sizeof 推算原陣列長度？
- [ ] 2. 函式指標如何實現 callback？呼叫時傳入的指標所指向的物件，需要存活到什麼時候？
- [ ] 3. gcc、make、git 與 uname -r 分別提供什麼資訊或功能？為什麼第 3 週要記錄正在執行的核心版本？
- [ ] 4. 短練習：挑教材中一個傳入指標或 callback 的小範例，標出呼叫者、參數、被指向的物件，以及物件生命週期；用 5–8 行筆記說明呼叫流程。

**選做實驗**：在 Linux 編譯小程式，加入 callback 並觀察傳入參數。

### 第 2 週｜核心常見 C 寫法

**必讀（依序開啟）**

- [ ] 1. [位元操作](https://hackmd.io/@sysprog/c-bitwise)：旗標、mask、shift。
- [ ] 2. [前置處理器](https://hackmd.io/@sysprog/c-preprocessor)：巨集展開、條件編譯與容易誤用的地方。
- [ ] 3. [C 記憶體與對齊](https://hackmd.io/@sysprog/c-memory)：只看 alignment、結構布局。
- [ ] 4. [未定義行為](https://hackmd.io/@sysprog/c-undefined-behavior)：挑整數、指標、生命週期相關案例。

**自我檢核（4 項）**

- [ ] 1. 如何用 mask 檢查、設定與清除某個旗標？使用位移運算前，要確認型別與位移量的哪些限制？
- [ ] 2. 巨集和函式在參數求值上有什麼差別？帶有副作用的參數，為什麼可能在巨集展開後造成問題？
- [ ] 3. 有號整數溢位與使用失效指標為什麼可能是未定義行為？為什麼一次執行結果正常，不能證明寫法正確？
- [ ] 4. 短練習：選一個包含 char 與 int 成員的 struct，畫出可能的成員、padding 與對齊布局；記下假設的型別大小及對齊值，說明為什麼 sizeof 可能大於成員大小總和。

**選做實驗**：寫小程式驗證容易記錯的觀念；某次執行成功不能證明未定義行為合法。

### 第 3 週｜第一個核心模組

**必讀（依序開啟）**

- [ ] 1. [LKMPG：核心標頭](https://sysprog21.github.io/lkmpg/#kernel-headers)：使用現有 Ubuntu 中正在執行的核心所對應的標頭，先核對 uname -r。
- [ ] 2. [LKMPG：Hello World](https://sysprog21.github.io/lkmpg/#hello-world)：固定從 hello-1 範例開始，讀 init／exit 與建置；參數、多檔模組選讀。
- [ ] 3. [LKMPG：使用者與核心空間](https://sysprog21.github.io/lkmpg/#user-space-vs-kernel-space)：先掌握模組執行的位置。
- [ ] 4. [LKMPG：常見限制](https://sysprog21.github.io/lkmpg/#common-pitfalls)：只看一般 C 程式庫與核心堆疊限制。

**自我檢核（4 項）**

- [ ] 1. 為什麼建置模組要使用與目標核心版本相符的標頭？uname -r、核心標頭與模組建置目錄之間有什麼關係？
- [ ] 2. hello-1 的初始化與退出入口各是哪個函式？載入成功、初始化回傳錯誤與卸載時，控制流程各有什麼差別？
- [ ] 3. 核心模組與一般使用者 C 程式的執行環境有什麼差異？為什麼不能直接使用一般 C 程式庫，或任意配置大型區域陣列？
- [ ] 4. 短練習：在 hello-1 原始碼中標出 init_module、cleanup_module 與訊息輸出；寫出成功載入到卸載的預期訊息順序，不必實際載入。

**選做實驗**：在 Ubuntu 實驗環境建置、載入、卸載 hello-1；保存 matching kernel headers 與核心版本。

### 第 4 週｜編譯、連結與啟動

**必讀（依序開啟）**

- [ ] 1. [連結器與執行檔](https://hackmd.io/@sysprog/c-linker-loader)：ELF、section、symbol、relocation 的基本關係。
- [ ] 2. [CRT](https://hackmd.io/@sysprog/c-runtime)：使用者程式如何進入 main。

**自我檢核（4 項）**

- [ ] 1. ELF 的 section 與 symbol 分別描述什麼？一個 .o 中尚未定義的 symbol，通常如何在連結時找到定義？
- [ ] 2. relocation 要修正什麼資訊？為什麼編譯器產生 .o 時，不能一律填入所有函式與資料的最終位址？
- [ ] 3. 一般使用者程式從入口點到 main，CRT 負責哪些準備？main 結束之後，還有哪些收尾流程？
- [ ] 4. 短練習：用教材中的兩個 C 檔案例子，畫出 C 原始碼 → .o → ELF 執行檔 → 載入 → main 的流程，標出跨檔函式的 symbol 與 relocation 在哪一階段處理。

**選做實驗**：用 readelf 與 nm 比較一般執行檔和 .ko，畫出編譯、連結、載入流程。

### 第 5 週｜系統呼叫、fork 與 exec

**必讀（依序開啟）**

- [ ] 1. [系統呼叫](https://hackmd.io/@sysprog/linux-syscall)：使用者空間進入核心的流程。
- [ ] 2. [fork／exec](https://hackmd.io/@sysprog/unix-fork-exec)：建立行程與替換程式映像。

**自我檢核（4 項）**

- [ ] 1. 函式庫函式與系統呼叫有什麼差別？為什麼一次函式庫呼叫，不能一律對應成一次同名系統呼叫？
- [ ] 2. fork 成功後，父行程與子行程如何從回傳值辨認自己的角色？兩者接著從哪裡繼續執行？
- [ ] 3. exec 與 fork 各改變什麼？exec 成功後，原本的程式碼與 PID 分別會怎樣？
- [ ] 4. 短練習：挑教材中的 fork／exec 範例，畫出父子行程的時間線；標出 fork 的兩個回傳值、exec 成功後不再執行的原程式碼，以及 exec 失敗時會走的路徑。

**選做實驗**：寫 fork／exec 小程式，用 strace 追蹤。

### 第 6 週｜行程與 CPU 排程

**必讀（依序開啟）**

- [ ] 1. [Process](https://hackmd.io/@sysprog/linux-process)：task_struct 概觀、行程狀態、thread、context switch。
- [ ] 2. [排程器](https://hackmd.io/@sysprog/linux-scheduler)：先看公平性、runqueue 與排程概觀；完整演算法留到第 37 週。

**自我檢核（4 項）**

- [ ] 1. Linux 如何用 task_struct 表示可被排程的工作？同一行程中的 threads 通常共享哪些資源，又各自保留哪些執行狀態？
- [ ] 2. runnable、running 與 blocked 有什麼差別？等待 I/O 的工作要經過哪些狀態，才會重新取得 CPU？
- [ ] 3. context switch 需要保存與恢復什麼？runqueue 為什麼主要關心可執行的工作，而不是所有存在的工作？
- [ ] 4. 短練習：畫出「A 執行 → A 等待 I/O → B 執行 → A 被喚醒 → A 再取得 CPU」的時間線，逐段標出 A／B 狀態；特別區分被喚醒與立即執行。

**選做實驗**：觀察 CPU 工作與等待 I/O 的程式，記錄狀態與排程差異。

## 第 7–12 週：記憶體與同步

### 第 7 週｜虛擬記憶體基礎

**必讀（依序開啟）**

- [ ] 1. [CS:APP 第 9 章導讀](https://hackmd.io/@sysprog/CSAPP-ch9)：虛擬位址、頁表、TLB、page fault、mmap。
- [ ] 2. [Linux 記憶體管理](https://hackmd.io/@sysprog/linux-memory)：只看位址映射與行程位址空間。

**自我檢核（4 項）**

- [ ] 1. 虛擬位址與實體位址有什麼差別？為什麼兩個行程可以使用相同數值的虛擬位址，卻存取不同資料？
- [ ] 2. 頁表與 TLB 各自負責什麼？TLB miss 與 page fault 為什麼不是同一件事？
- [ ] 3. mmap 建立的是什麼映射？page fault 可能是正常的記憶體管理流程，也可能是非法存取，兩者如何區分？
- [ ] 4. 短練習：畫出一次虛擬位址存取的簡化流程，分別標出 TLB 命中、需要查頁表與缺頁處理的分支；不必畫完整多層頁表。

**選做實驗**：從 /proc/<pid>/maps 辨認程式碼、函式庫、heap 與 stack。

### 第 8 週｜延遲配置與 Copy on Write

**必讀（依序開啟）**

- [ ] 1. [Linux 記憶體管理](https://hackmd.io/@sysprog/linux-memory)：page fault、lazy allocation、overcommit；先區分延遲配置與 COW。
- [ ] 2. [COW 教材](https://hackmd.io/@linD026/Linux-kernel-COW-content)：追概念與主要流程，暫不逐行讀完實作。

**自我檢核（4 項）**

- [ ] 1. lazy allocation 與 Copy on Write 各延後了哪一件工作？為什麼不能把兩者視為同一種機制？
- [ ] 2. fork 後父子行程暫時共享實體頁面時，核心如何避免其中一方寫入而直接改到另一方的私人資料？
- [ ] 3. 成功取得一段虛擬記憶體，是否表示全部實體頁面都已配置？overcommit 與之後真正使用記憶體的風險有什麼關係？
- [ ] 4. 短練習：畫出 fork 前、fork 後未寫入、子行程寫入一頁後的三張映射圖；標出各行程的虛擬頁、共享實體頁及新複製的頁面。

**選做實驗**：比較 fork 前後與寫入後的記憶體指標；將觀察結果與機制對照。

### 第 9 週｜核心配置與使用者資料交換

**必讀（依序開啟）**

- [ ] 1. [LKMPG：執行脈絡入門](https://sysprog21.github.io/lkmpg/#user-space-vs-kernel-space)：只看 process／softirq／hardirq 與可否睡眠的表格；完整中斷留到第 16 週。
- [ ] 2. [LKMPG：配置與所有權](https://sysprog21.github.io/lkmpg/#memory-allocation-and-ownership)：配置、釋放、錯誤路徑、kmalloc／vmalloc；特別看 GFP_KERNEL 可能睡眠。
- [ ] 3. [LKMPG：跨 user／kernel 複製](https://sysprog21.github.io/lkmpg/#copying-data-across-the-userkernel-boundary)：copy_to_user／copy_from_user 可能觸發缺頁、睡眠，不能放在 hardirq 或持有 spinlock 的區段。

**自我檢核（4 項）**

- [ ] 1. kmalloc 與 vmalloc 在虛擬及實體連續性上有什麼差別？選擇配置方式時，還要考慮哪些使用限制？
- [ ] 2. GFP_KERNEL 配置與 copy_to_user／copy_from_user 為什麼可能睡眠？為什麼不能在 hardirq 或持有 spinlock 的區段呼叫？
- [ ] 3. 配置、使用與釋放核心緩衝區時，誰負責持有及清理資源？中途配置或資料複製失敗時，錯誤路徑要處理什麼？
- [ ] 4. 短練習：挑教材的一段緩衝區配置與 usercopy 範例，逐行標出可能失敗、可能睡眠及需要釋放資源的位置，畫出成功與失敗兩條路徑。

**選做實驗**：在模組中配置並釋放緩衝區，補上配置失敗的清理路徑。

### 第 10 週｜共享記憶體

**必讀（依序開啟）**

- [ ] 1. [共享記憶體整理](https://hackmd.io/@sysprog/linux-shared-memory)：各種機制的差異。
- [ ] 2. [POSIX shared memory 範例](https://logan.tw/posts/2018/01/07/posix-shared-memory/)：讀使用流程與範例，執行列為選做。

**自我檢核（4 項）**

- [ ] 1. POSIX shared memory 範例中，建立物件、設定大小與 mmap 各自完成什麼工作？兩個行程如何找到同一份共享資料？
- [ ] 2. munmap、close 與 shm_unlink 各清理什麼？為什麼只關閉 fd，不能當成所有共享記憶體清理工作都已完成？
- [ ] 3. 資料已經共享，為什麼仍可能需要同步？兩個行程同時讀寫同一個欄位，會產生哪些不一致的情況？
- [ ] 4. 短練習：沿現成範例列出建立 → 設定大小 → 映射 → 讀寫 → 解除映射 → 清理的步驟，註記每個步驟由哪個行程執行，以及它操作的物件或 fd。

**選做實驗**：執行 POSIX shared memory 範例，讓兩個行程共享資料；先記錄存取問題。

### 第 11 週｜並行、競爭與 mutex

**必讀（依序開啟）**

- [ ] 1. [並行系列入口](https://hackmd.io/@sysprog/concurrency)：只選概念、執行順序、資料競爭與 atomics 的部分。
- [ ] 2. [Mutex 原理與實作](https://hackmd.io/@sysprog/concurrency-mutex)：理解等待與互斥；futex 深入實作可後補。

**自我檢核（4 項）**

- [ ] 1. 兩個執行緒對共享計數器做 ++，為什麼可能丟失更新？讀取、計算與寫回之間會出現什麼交錯？
- [ ] 2. mutex 如何保護 critical section？為什麼讀取與寫入共享狀態的相關路徑，都需要遵守同一套加鎖規則？
- [ ] 3. 單次 atomic 操作與一整段操作的互斥有什麼差別？遇到鎖已被持有時，等待和持續忙等各會帶來什麼影響？
- [ ] 4. 短練習：寫出兩個執行緒把計數器從 0 各增加一次、最後卻只得到 1 的交錯表；再圈出必須一起受到 mutex 保護的步驟。

**選做實驗**：比較共享計數器加鎖前後；自製 mutex 或深入 futex 可延後。

### 第 12 週｜核心中的同步

**必讀（依序開啟）**

- [ ] 1. [Linux 同步](https://hackmd.io/@sysprog/linux-sync)：執行脈絡、睡眠與鎖的基本關係。
- [ ] 2. [LKMPG：mutex](https://sysprog21.github.io/lkmpg/#mutex)。
- [ ] 3. [LKMPG：spinlock](https://sysprog21.github.io/lkmpg/#spinlocks) → [atomic 操作](https://sysprog21.github.io/lkmpg/#atomic-operations)：先看不可睡眠的限制，不在鎖內做 usercopy／GFP_KERNEL 配置；atomic 計數不等於完整同步協定。

**自我檢核（4 項）**

- [ ] 1. 核心 mutex 與 spinlock 的等待方式有什麼差別？選鎖時，為什麼要先確認呼叫脈絡及能否睡眠？
- [ ] 2. 如果同一份資料也會被中斷路徑存取，選鎖與 IRQ 控制時要檢查哪些條件？為什麼不能只看平常的行程路徑？
- [ ] 3. 為什麼把計數器改成 atomic，不代表相關緩衝區與狀態欄位就一起受到保護？哪些跨欄位條件仍需同步？
- [ ] 4. 短練習：把「讀寫共享欄位、GFP_KERNEL 配置、copy_to_user」排成一段流程，標出 spinlock 可保護的短區段，以及必須移到鎖外的操作，寫下理由。

**選做實驗**：為既有模組緩衝區選同步方式並說明理由；RCU 與完整 memory ordering 留到後段。

## 第 13–19 週：裝置介面與事件

### 第 13 週｜字元裝置讀寫

**必讀（依序開啟）**

- [ ] 1. [CS:APP 第 10 章：I/O 入門](https://hackmd.io/@sysprog/CSAPP-ch10)：先看 fd 與 open／read／write／close；本週只看基本使用者介面。
- [ ] 2. [補充／系統手冊：read](https://man7.org/linux/man-pages/man2/read.2.html) → [write](https://man7.org/linux/man-pages/man2/write.2.html)：只看 DESCRIPTION／RETURN VALUE，理解 short read／write，以及非零大小讀取回傳 0 的 EOF 意義。
- [ ] 3. [LKMPG：字元裝置](https://sysprog21.github.io/lkmpg/#character-device-drivers)：固定追 chardev.c，對照 file_operations、open／release、read、註冊與移除；自加 write 列為選做。
- [ ] 4. [LKMPG：user／kernel 資料交換](https://sysprog21.github.io/lkmpg/#copying-data-across-the-userkernel-boundary)：在裝置範例中回查。

**自我檢核（4 項）**

- [ ] 1. 使用者程式的 fd 與 read 呼叫，如何連到 chardev.c 的 file_operations callback？open／release 與裝置註冊／移除各扮演什麼角色？
- [ ] 2. 使用者空間的 read／write 回傳正數或 -1 各表示什麼？非零大小的 read 回傳 0，與 write 回傳 0 的意義有何不同？為什麼成功時仍可能只處理部分 bytes？
- [ ] 3. 驅動為什麼要使用 usercopy 介面交換資料？chardev.c 如何決定回傳的資料量，以及後續讀取是否到達 EOF？
- [ ] 4. 短練習：沿 chardev.c 畫出 open → 第一次 read → 後續 read → close 的呼叫流程，標出每次回傳值與 EOF 判斷；只追現成 read，不必新增 write。

**選做實驗**：先執行現成 chardev，再接上自己的緩衝區與 write；自行整合可延長。

### 第 14 週｜阻塞與 poll

**必讀（依序開啟）**

- [ ] 1. [I/O 模型入門](https://hackmd.io/@sysprog/linux-io-model)：從第 21 週提前，先看 blocking、nonblocking 與 readiness；完整事件迴圈留到第 21 週。
- [ ] 2. [補充／系統手冊：open](https://man7.org/linux/man-pages/man2/open.2.html)：只看 O_NONBLOCK 段落；回查上一週 read／write 的 ERRORS 中 EAGAIN／EWOULDBLOCK。
- [ ] 3. [LKMPG：等待與喚醒](https://sysprog21.github.io/lkmpg/#blocking-processes-and-threads)：固定追 Sleep 的範例，找出條件與 wakeup。
- [ ] 4. [LKMPG：poll／select／epoll](https://sysprog21.github.io/lkmpg/#poll-select-and-epoll)：對照 wait queue 與 readiness mask；readiness 不代表下一次讀取一定取得全部資料。

**自我檢核（4 項）**

- [ ] 1. 支援非阻塞 I/O 的裝置或 socket 暫時沒有資料時，blocking 與 O_NONBLOCK 路徑各應如何處理？EAGAIN 表示什麼？
- [ ] 2. wait queue、等待條件與 wakeup 各負責什麼？為什麼被喚醒後仍要重新檢查條件？
- [ ] 3. poll 回報 readiness，和實際完成一次 read 有什麼差別？為什麼 readiness 不保證取得全部資料，也不能忽略非阻塞讀取的結果？
- [ ] 4. 短練習：畫出「讀者檢查條件 → 等待 → 寫者更新狀態並喚醒 → 讀者重查條件」流程，另外標出 O_NONBLOCK 與 poll 各在哪裡分支。

**選做實驗**：在現成等待範例上追喚醒；有時間再為自己的裝置加入 poll。

### 第 15 週｜處理器與 cache

**必讀（依序開啟）**

- [ ] 1. [處理器基礎](https://hackmd.io/@sysprog/cpu-basics)：只看指令執行、cache、記憶體階層與多核概觀。
- [ ] 2. [CS:APP 第 6 章導讀](https://hackmd.io/@sysprog/CSAPP-ch6)：locality 與 cache。

**自我檢核（4 項）**

- [ ] 1. 時間區域性與空間區域性各是什麼？它們如何影響 cache 命中與資料存取成本？
- [ ] 2. CPU cache 與 TLB 儲存的資訊有什麼差別？為什麼資料已在 cache，仍不能直接推論位址轉換一定命中 TLB？
- [ ] 3. 連續與跨步存取同一個陣列，為什麼可能有不同效能？除了存取模式，資料量與 cache line 大小會如何影響比較？
- [ ] 4. 短練習：假設 cache line 為 64 bytes、每個 int 為 4 bytes，且 a[0] 位於 cache line 起點，畫出 a[0]、a[1] 與 a[16] 所在的 cache line；再對照連續及每次跨 16 個元素的存取。

**選做實驗**：比較連續與跨步存取的耗時，記錄資料量與測試條件。

### 第 16 週｜中斷與 timer

**必讀（依序開啟）**

- [ ] 1. [中斷處理](https://hackmd.io/@sysprog/linux-interrupt)：中斷流程與不同執行脈絡。
- [ ] 2. [時間機制](https://hackmd.io/@sysprog/linux-timer)：先看 jiffies、timer、hrtimer 概觀。

**自我檢核（4 項）**

- [ ] 1. 在本週的一般非 PREEMPT_RT 脈絡下，hardirq、softirq 與 process context 有哪些睡眠限制？為什麼即使在 process context，也仍需檢查是否持有 spinlock？
- [ ] 2. 中斷處理為什麼通常要盡快完成？需要較久或可能睡眠的工作，應如何拆成立即處理與延後處理？
- [ ] 3. jiffies、一般 timer 與 hrtimer 分別用來表達或處理什麼？設定到期時間，為什麼不等於保證工作能在該時刻完成？
- [ ] 4. 短練習：挑教材的一個中斷或 timer 範例，畫出事件發生到 callback 的流程，標出執行脈絡與需要延後的工作；沒有硬體也可只追原始碼。

**選做實驗**：追一個 timer 或中斷範例；硬體相關範例以閱讀為主。

### 第 17 週｜延後工作與事件整合

**必讀（依序開啟）**

- [ ] 1. [LKMPG：時間與延後執行](https://sysprog21.github.io/lkmpg/#timekeeping-and-deferred-execution)。
- [ ] 2. [LKMPG：workqueue](https://sysprog21.github.io/lkmpg/#work-queues)。
- [ ] 3. [LKMPG：等待與喚醒](https://sysprog21.github.io/lkmpg/#blocking-processes-and-threads)：回查整合所需部分。

**自我檢核（4 項）**

- [ ] 1. 排程 work 與實際執行 work callback 有什麼差別？為什麼呼叫 schedule_work 不代表工作已在當下完成？
- [ ] 2. 一般 timer callback 與一般 workqueue callback 的執行脈絡有什麼差別？這會如何影響可否睡眠及鎖的選擇？
- [ ] 3. 模組退出前，為什麼要停止新的事件並取消或等待尚未完成的 timer／work？否則可能碰到什麼生命週期問題？
- [ ] 4. 短練習：畫出 timer 觸發 → 排程 work → work 更新資料 → wakeup 讀者的流程；再加一條卸載路徑，標出必須停止與等待的工作。

**選做實驗**：用 timer／workqueue 模擬資料到達，喚醒讀者；檢查卸載清理。

### 第 18 週｜device、driver、bus 與 sysfs

**必讀（依序開啟）**

- [ ] 1. [原課程驅動模型教材](https://github.com/gregkh/presentation-driver-model)：先看投影片概觀。
- [ ] 2. [LKMPG：Device Model](https://sysprog21.github.io/lkmpg/#standardizing-the-interfaces-the-device-model)。
- [ ] 3. [LKMPG：sysfs](https://sysprog21.github.io/lkmpg/#sysfs-interacting-with-your-module)。

**自我檢核（4 項）**

- [ ] 1. device、driver 與 bus 各描述什麼？核心如何透過 bus 的配對機制，把裝置交給適合的 driver？
- [ ] 2. sysfs 如何呈現裝置、driver 與屬性？讀寫一個 sysfs 屬性，和直接存取裝置硬體暫存器有什麼差別？
- [ ] 3. 裝置或 driver 移除時，為什麼要同時考慮屬性介面、未完成操作與資源生命週期？probe／remove 如何對應資源取得與清理？
- [ ] 4. 短練習：挑教材中的一個裝置模型例子，畫出 device、driver、bus 與 sysfs 的關係；若 Ubuntu 已開啟，也可改用現有 sysfs 裝置辨認這些關係。

**選做實驗**：在 sysfs 中追蹤一個現有裝置與所使用的 driver。

### 第 19 週｜Device Tree

**必讀（依序開啟）**

- [ ] 1. [Device Tree 入門投影片](https://events.static.linuxfound.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf)。
- [ ] 2. [LKMPG：Device Tree](https://sysprog21.github.io/lkmpg/#device-tree)：節點、compatible、資源、模組配對；測試範例可分次完成。

**自我檢核（4 項）**

- [ ] 1. DTS 範例中的節點名稱、compatible 與 reg 各描述什麼，它們分別如何連到裝置配對與資源取得？
- [ ] 2. driver 的 of_match_table 如何與 compatible 配對，配對成功之後何時才會呼叫 probe？
- [ ] 3. 讀取 reg 的位址與長度時，為什麼需要一起查看父節點的 #address-cells 與 #size-cells？
- [ ] 4. 短練習：選教材中一個 DTS 裝置節點，逐項標註 compatible、reg 與父節點的 cells，畫出它與 match table、probe 的連接；只閱讀，不必建置平台。

**選做實驗**：使用教材的平台／QEMU 範例驗證 DTS；平台建置可另留時間。

## 第 20–24 週：網路、觀測與整合

### 第 20 週｜socket 與封包路徑

**必讀（依序開啟）**

- [ ] 1. [補充／系統手冊：socket](https://man7.org/linux/man-pages/man2/socket.2.html)：只看 socket type 與基本使用流程，先區分 datagram 與 stream。
- [ ] 2. [補充／固定範例：UDP echo client／server](https://man7.org/linux/man-pages/man3/getaddrinfo.3.html#EXAMPLES)：先看 hints／addrinfo，再依序追 EXAMPLES 的 server 與 client。這是 UDP 範例，沒有 TCP 的 listen／accept 流程；本週不額外自行選材。
- [ ] 3. [核心網路：socket layer](https://hackmd.io/@rickywu0421/linux_networking_1)：只讀 socket 與核心結構連結的概觀，其餘延到第二輪。

**自我檢核（4 項）**

- [ ] 1. 固定 UDP 範例在哪裡選定 SOCK_DGRAM，datagram 與 stream 對訊息邊界的處理有什麼差異？
- [ ] 2. server 與 client 的 getaddrinfo hints 有哪些差異，為什麼都要逐一嘗試回傳的 addrinfo？
- [ ] 3. server 的 bind、recvfrom／sendto 與 client 的 connect、write／read 各負責什麼，UDP client 呼叫 connect 是否代表執行 TCP 的連線流程？
- [ ] 4. 短練習：沿固定 UDP 範例畫一張雙欄流程圖，左欄 server、右欄 client，標出位址解析、socket 建立、收送位置及傳遞的位址；不必執行程式。

**選做實驗**：執行手冊的 UDP server 與 client，在本機 127.0.0.1 傳送訊息；TCP 另作延伸。

### 第 21 週｜事件驅動 I/O

**必讀（依序開啟）**

- [ ] 1. [I/O 模型演進](https://hackmd.io/@sysprog/linux-io-model)：第二輪閱讀，接第 14 週，聚焦 socket 多工、事件迴圈與通知；不用重讀已懂的阻塞概念。
- [ ] 2. [LKMPG：poll／select／epoll](https://sysprog21.github.io/lkmpg/#poll-select-and-epoll)：與第 14 週對照。

選讀：第 20 週的 [UDP echo 範例](https://man7.org/linux/man-pages/man3/getaddrinfo.3.html#EXAMPLES) 可用來思考如何加入事件迴圈；[封包處理投影片](https://www.slideshare.net/DevopsCon/how-linux-processes-your-network-packet-elazar-leibovich) 與 [PACKET_MMAP 概觀](https://www.kernel.org/doc/Documentation/networking/packet_mmap.txt) 留作網路深入閱讀。

**自我檢核（4 項）**

- [ ] 1. 在單一執行緒處理多個 socket 時，直接對其中一個 blocking socket 呼叫 recvfrom，會如何影響其他 socket？
- [ ] 2. poll／epoll 回報可讀與真正取得資料之間有什麼差別，為什麼非阻塞讀取仍需要處理 EAGAIN？
- [ ] 3. driver 的 poll callback 中，poll_wait 與 readiness mask 分別做什麼，它們如何連到使用者端的等待？
- [ ] 4. 短練習：替第 20 週 UDP server 寫一段不需編譯的事件迴圈偽碼，標出等待 readiness、非阻塞接收、EAGAIN 與回覆的位置；最多 15 行。

**選做實驗**：把 UDP 範例改成 poll 或 epoll；封包擷取與 PACKET_MMAP 留作延伸。

### 第 22 週｜eBPF 與除錯入口

**必讀（依序開啟）**

- [ ] 1. [eBPF 觀測](https://hackmd.io/@sysprog/linux-ebpf)：只讀一個現成系統呼叫／I/O 觀測案例，不要求自行開發 eBPF 程式。

選讀：[panic 判讀](https://wiki.csie.ncku.edu.tw/kernel-debugging.pdf) 的錯誤訊息與呼叫堆疊；[LKMPG debugfs](https://sysprog21.github.io/lkmpg/#debugfs-flexible-debugging-interfaces) 的模組除錯介面。需要時再補，不和 eBPF 同週全部完成。

**自我檢核（4 項）**

- [ ] 1. 你選的 eBPF 觀測案例掛在哪一類事件上，哪個操作會觸發它，資料是在核心端還是使用者端整理？
- [ ] 2. 案例輸出的每個主要欄位來自哪裡，有沒有 PID、事件名稱或時間等能用來區分事件的資訊？
- [ ] 3. 從這個案例能回答哪一個具體系統行為問題，僅靠現有輸出又有哪些事情不能判定？
- [ ] 4. 短練習：在選定案例的程式或教材輸出旁標註事件來源與欄位含義，再寫一個可用該輸出驗證的問題；環境未就緒時直接使用教材範例。

**選做實驗**：執行一個 eBPF 案例；環境未就緒時先以 strace／核心 log 觀察。panic 與 debugfs 選讀。

### 第 23 週｜共同專題整合

**必讀（依序開啟）**

- [ ] 1. [LKMPG：字元裝置](https://sysprog21.github.io/lkmpg/#character-device-drivers)：回查 read／write。
- [ ] 2. [LKMPG：等待與喚醒](https://sysprog21.github.io/lkmpg/#blocking-processes-and-threads)：回查阻塞行為。
- [ ] 3. [LKMPG：同步](https://sysprog21.github.io/lkmpg/#synchronization)：回查共享資料保護。

**自我檢核（4 項）**

- [ ] 1. 在讀取端等待資料、寫入端提供資料的流程中，共享狀態由誰更新，read／write 哪些區段需要互斥？
- [ ] 2. wait queue 的等待條件與 wake-up 各負責什麼，讀取端被喚醒後為什麼仍要重新確認條件？
- [ ] 3. 同一個裝置沒有資料時，blocking read 與 O_NONBLOCK read 應如何回應，這和 poll 的可讀狀態如何一致？
- [ ] 4. 短練習：用讀過的字元裝置與等待範例畫出一次『讀取等待 → 寫入資料 → 喚醒 → 讀取完成』，標上條件檢查、共享資料保護與 usercopy；不必整合新模組。

**選做實驗**：整合自己的記憶體型裝置，測試多讀寫者、非阻塞與錯誤路徑。

### 第 24 週｜測試、清理與筆記

**必讀（依序開啟）**

- [ ] 1. [LKMPG：配置與所有權](https://sysprog21.github.io/lkmpg/#memory-allocation-and-ownership)：回查釋放流程。
- [ ] 2. [LKMPG：時間與延後執行](https://sysprog21.github.io/lkmpg/#timekeeping-and-deferred-execution)：回查取消與卸載。

**自我檢核（4 項）**

- [ ] 1. 讀過的配置範例有哪些資源需要釋放，它們的持有者是誰，成功與中途失敗的清理路徑有何不同？
- [ ] 2. 若 timer callback 或 workqueue 仍可能存取模組資料，卸載時直接釋放資料會產生什麼問題？
- [ ] 3. 取消延後工作、等待執行結束與釋放資料之間應如何安排，如何避免 callback 在清理期間再次排入工作？
- [ ] 4. 短練習：選一個配置或延後工作範例，列出最多 6 個取得／取消／釋放步驟；再選一個中途失敗點，畫出已取得資源的回收順序。

**選做實驗**：保存重現步驟與測試結果，從乾淨 VM 重新操作自己的專題。

## 第 25–30 週：驅動深入

本階段先讀 LKMPG 的 PCI、MMIO、IRQ、DMA 介面，再用 QEMU EDU 的同一份規格對照事件與資料路徑。EDU 規格不提供完整 Linux 驅動教程；理解圖與 API 追蹤是基本完成標準，自己整合 EDU 驅動列為選做。

需要操作 EDU 時，以現有 Ubuntu 作為 Linux 建置端，使用 LKMPG 的專用 QEMU 核心與 BusyBox。它和第 3 週的 Ubuntu 核心是兩個不同目標；對應的核心、標頭與 .ko 不要混用。

### 第 25 週｜裝置資源與 PCI 配對

**必讀（依序開啟）**

- [ ] 1. [LKMPG：QEMU 實驗環境](https://sysprog21.github.io/lkmpg/#qemubased-development-environment)：讀 Linux 建置端、核心／BusyBox、build 與 boot 的流程；建置是選做，可另留時間。
- [ ] 2. [LKMPG：PCI 驅動](https://sysprog21.github.io/lkmpg/#pci-drivers)：配對、probe／remove、資源管理。
- [ ] 3. [補充／官方：EDU 裝置規格](https://www.qemu.org/docs/master/specs/edu.html)：先讀啟動選項、PCI 規格與 MMIO 概觀。

**自我檢核（4 項）**

- [ ] 1. EDU 的 vendor／device ID 與 BAR0 規格是什麼，driver 的 match table 與資源查詢各使用哪一部分？
- [ ] 2. pci_driver 註冊、PCI 裝置配對、probe 與 remove 分別何時發生，為什麼不能把模組載入等同於 probe 一定成功？
- [ ] 3. Linux 建置端、QEMU guest、guest kernel headers 與 .ko 各位於哪一層，為什麼不能混用 Ubuntu 核心與專用 QEMU 核心的建置產物？
- [ ] 4. 短練習：依教材畫出『Ubuntu 建置端 → 專用 guest kernel／BusyBox → QEMU 加入 EDU → PCI 配對 → probe』，旁註 ID 與 BAR0；不必下載或建置。

**選做實驗**：準備專用 QEMU 環境並加入 EDU，從 guest 找到裝置；環境建置可另留一個單元。

### 第 26 週｜MMIO 與暫存器

**必讀（依序開啟）**

- [ ] 1. [LKMPG：MMIO](https://sysprog21.github.io/lkmpg/#memorymapped-io-and-register-access)：資源取得、映射、readl／writel。
- [ ] 2. [補充／官方：裝置 I/O](https://docs.kernel.org/driver-api/device-io.html)：只讀 Memory Mapped IO 與 __iomem pointer tokens，先補 accessor、posted write、需要時的 read-back。
- [ ] 3. [補充／官方：EDU](https://www.qemu.org/docs/master/specs/edu.html)：本週只讀 MMIO area spec 與 liveness check。

**自我檢核（4 項）**

- [ ] 1. BAR 中的裝置資源、映射後的 __iomem token 與一般 RAM 指標有何差異，為什麼 MMIO 要透過 readl／writel 等 accessor？
- [ ] 2. 依 EDU 規格，liveness register 的位移、存取寬度與讀寫行為是什麼，如何選擇對應 accessor？
- [ ] 3. MMIO 寫入已被 CPU 發出與裝置已收到寫入有何差別，在需要確認 posted write 生效時，文件說明的 read-back 起什麼作用？
- [ ] 4. 短練習：替 EDU liveness check 寫最多 10 行偽碼，標出取得 BAR、映射、寫入、讀回與比較，並註記使用的 register offset 與寬度；不必做完整驅動。

**選做實驗**：實作 EDU liveness check；若只閱讀，畫出 BAR → mapping → readl／writel 的流程即可。

### 第 27 週｜裝置中斷與延後處理

**必讀（依序開啟）**

- [ ] 1. [LKMPG：中斷 handler](https://sysprog21.github.io/lkmpg/#interrupt-handlers)。
- [ ] 2. [補充／官方：EDU](https://www.qemu.org/docs/master/specs/edu.html)：中斷狀態、raise／acknowledge、IRQ controller。
- [ ] 3. [LKMPG：workqueue](https://sysprog21.github.io/lkmpg/#work-queues)：對照第 17 週。

**自我檢核（4 項）**

- [ ] 1. EDU 的 interrupt status、raise 與 acknowledge registers 分別做什麼，讀取 status 是否就等同於清除中斷？
- [ ] 2. EDU 使用 INTx 或 MSI 時，raise 與 acknowledge 的規格有哪些共同要求，driver 應在何處完成 acknowledge？
- [ ] 3. handler 中哪些工作必須即時完成，哪些可交給 workqueue，為什麼能否睡眠會影響這個分工？
- [ ] 4. 短練習：畫出『EDU raise → 核心 handler → 讀 status／ack → 排入 workqueue → 通知等待者』，逐段標註執行脈絡；只對照介面，不必新增 IRQ 驅動。

**選做實驗**：在既有驅動中接 IRQ 通知；需要自行補驅動程式，不作為本週必達成果。

### 第 28 週｜DMA

**必讀（依序開啟）**

- [ ] 1. [LKMPG：DMA 與 mmap](https://sysprog21.github.io/lkmpg/#dma-and-mmap)：DMA mask、addressing、coherent／streaming、ownership；本週先不做 mmap。
- [ ] 2. [補充／官方：DMA HOWTO](https://docs.kernel.org/core-api/dma-api-howto.html)：只讀 CPU／DMA address、mask、Types of DMA mappings；特別看 coherent memory 仍可能需要 ordering 保證。
- [ ] 3. [補充／官方：EDU](https://www.qemu.org/docs/master/specs/edu.html)：DMA registers 與 controller；注意預設 28-bit DMA mask，依自己的 QEMU 設定核對。
- [ ] 4. [補充／固定範例：LKMPG dma.c](https://github.com/sysprog21/lkmpg/blob/5854698bc4535357d9c40d4eefa5241ef3bd814e/examples/dma.c)：只追 mask、coherent／streaming mapping、probe 與 cleanup。這是 synthetic platform device 的 API 示範，不是 EDU 驅動，也不操作真實 DMA 硬體；閱讀快照固定於此 commit。

選讀／建置入口：[同版本的 examples Makefile](https://github.com/sysprog21/lkmpg/blob/5854698bc4535357d9c40d4eefa5241ef3bd814e/examples/Makefile)，核對 dma.o 與 KDIR；建置時對齊目標核心。

**自我檢核（4 項）**

- [ ] 1. 固定 dma.c 中 coherent allocation 回傳的 CPU pointer 與 DMA handle 各給誰使用，DMA address 為什麼不能直接當成 CPU physical address？
- [ ] 2. coherent 與 streaming mappings 在配置、同步與 ownership 上有什麼差別，coherent memory 為什麼仍可能需要 ordering 保證？
- [ ] 3. dma.c 的 mask 與 EDU 預設 mask 各是多少，為什麼要在配置／mapping 前設定 mask，這個 synthetic 範例有沒有實際啟動裝置資料傳輸？
- [ ] 4. 短練習：在固定 dma.c 中標出 set mask、coherent allocation、streaming mapping、mapping error 檢查、unmap 與 cleanup；整理一張資源生命週期圖，不必建置或操作 EDU。

**選做實驗**：依 matching headers 建置 dma.c，觀察 API 生命週期。EDU 的真實資料傳輸另作較大延伸，不能用此範例代替。

### 第 29 週｜多核、排序與物件生命週期

**必讀（依序開啟）**

- [ ] 1. [多核與 spinlock](https://hackmd.io/@sysprog/multicore-locks)：重點看共享資料與鎖的成本。
- [ ] 2. [記憶體模型](https://mes0903.github.io/memory/memory_model/)：先看 ordering 的概念與範例。
- [ ] 3. [Reference Count 案例](https://hackmd.io/@sysprog/concurrency-reference-count)：物件何時可以釋放。

**自我檢核（4 項）**

- [ ] 1. 互斥、操作的 atomicity 與存取 ordering 各解決什麼問題，為什麼不能把『一次操作不可分割』當成『所有存取都按原始碼順序被觀察到』？
- [ ] 2. 多核反覆爭用同一把 spinlock 時，等待、cache line 移動與 critical section 長度如何影響成本？
- [ ] 3. 引用計數如何決定物件是否仍被使用，僅把計數加減改成 atomic，是否就能同時保證物件欄位存取與釋放安全？
- [ ] 4. 短練習：從本週選一個小型共享物件案例，畫出兩個使用者取得／釋放引用的時序，標出資料存取、最後釋放與可能需要鎖或 ordering 的位置。

**選做實驗**：檢視自己的模組／驅動，標出共享狀態、事件通知與卸載需要的保護。

### 第 30 週｜驅動階段收尾

**必讀（依序開啟）**

- [ ] 1. [LKMPG：Device Model](https://sysprog21.github.io/lkmpg/#standardizing-the-interfaces-the-device-model)：回查資源與生命週期。
- [ ] 2. [LKMPG：DMA 常見錯誤](https://sysprog21.github.io/lkmpg/#dma-debugging-and-common-mistakes)。

**自我檢核（4 項）**

- [ ] 1. 在 PCI 配對 → MMIO → IRQ → DMA 的路徑中，哪些資訊來自 EDU 規格，哪些責任由 Linux driver API 處理？
- [ ] 2. 若 probe 在部分資源取得後失敗，需要回收哪些已取得資源，手動管理與 managed resources 的清理責任何處不同？
- [ ] 3. 移除裝置或卸載驅動時，為什麼必須先處理仍在進行的 I/O、DMA 或延後工作，才能釋放其使用的記憶體？
- [ ] 4. 短練習：畫一張驅動流程總圖，選一個『映射失敗』或『DMA mapping 失敗』分支，列出回收順序；把僅追讀與已實作的部分分別標出。

**選做實驗**：對已完成的功能測試載入／卸載與錯誤路徑；尚未完成的 DMA 不必硬湊成完整驅動。

## 第 31–36 週：虛擬化

固定案例使用 [kvm-host 閱讀快照](https://github.com/sysprog21/kvm-host/tree/75962e148b3e60c75290dfe7b64bd1d54354cb87)，先讀同版本 README 的 Build and Run，再沿著程式追 VM 建立、記憶體註冊、vCPU 執行與 exit；KVM API 留作對照查表。virtio、網路、磁碟、seccomp 的完整實作為選讀，不要求自己寫通用 VMM。

閱讀範例已固定 commit；執行前仍需依該 README 準備 toolchain、guest kernel 與 /dev/kvm，不能直接假設沿用前面的 guest image。這份課表未在你的 Linux／KVM 環境測試範例建置，實跑可另留時間。

KVM 實驗需要 Linux 能存取 /dev/kvm；若 Linux 本身在 VM 中，需確認 nested virtualization。QEMU 可先用軟體模擬進行環境練習；KVM 環境不具備時先追範例，不以執行成功作為閱讀進度門檻。

### 第 31 週｜QEMU 與 KVM 的分工

**必讀（依序開啟）**

- [ ] 1. [原課程 KVM 教材](https://hackmd.io/@sysprog/linux-kvm)：架構概觀與硬體支援。
- [ ] 2. [補充／官方：QEMU 系統模擬](https://www.qemu.org/docs/master/system/index.html)：從 Introduction 讀起，重點看 accelerators。

**自我檢核（4 項）**

- [ ] 1. QEMU 使用 TCG 與 KVM accelerator 時，guest 指令由誰執行，KVM 模式中的 CPU 與記憶體虛擬化由哪一層提供？
- [ ] 2. 當 guest 需要存取模擬周邊時，QEMU、KVM 與 guest driver 各負責哪一段，為什麼只有 KVM 並不等於具備所有裝置模型？
- [ ] 3. Linux 本身在另一個 VM 內時，實驗中的 host 與 guest 分別是誰，nested virtualization 與 /dev/kvm 可用性如何影響後續實跑？
- [ ] 4. 短練習：畫出一般 Linux host 與『Linux host 本身在 VM 中』兩種層級圖，標出 QEMU、KVM、guest driver 與硬體；不必檢查或修改機器設定。

**選做實驗**：檢查 /dev/kvm 是否可用，記錄是否處於 nested virtualization。

### 第 32 週｜建立可觀察的 guest

**必讀（依序開啟）**

- [ ] 1. [補充／官方：QEMU 系統模擬](https://www.qemu.org/docs/master/system/index.html)：選 Invocation 中的 CPU、memory、disk、console 與 accelerator 選項。
- [ ] 2. [LKMPG：QEMU 環境](https://sysprog21.github.io/lkmpg/#qemubased-development-environment)：回用已有環境。

**自我檢核（4 項）**

- [ ] 1. 教材的 QEMU 啟動命令中，CPU 型號／數量、memory、disk 與 console 分別由哪些選項控制？
- [ ] 2. guest kernel、root filesystem 與磁碟映像各扮演什麼角色，使用 -kernel 與傳統從磁碟開機的入口有何不同？
- [ ] 3. 從啟動選項如何辨認 TCG 或 KVM 模式，為什麼不能僅憑 guest 成功開機就判定已使用 KVM？
- [ ] 4. 短練習：複製教材的一條 QEMU 啟動命令到筆記，逐項標註用途；再寫出只切換 accelerator 的兩個版本，不必執行或重新建置 guest。

**選做實驗**：啟動現成 guest，保存命令與版本；沒有 KVM 時先用軟體模擬。

### 第 33 週｜KVM 的 fd 與 ioctl

**必讀（依序開啟）**

- [ ] 1. [補充／固定範例：kvm-host README](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/README.md)：讀 Build and Run，再開下一個程式連結。
- [ ] 2. [補充／固定範例：vm.c](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/src/vm.c)：本週只追 vm_init 的 /dev/kvm 與 VM／vCPU 建立。
- [ ] 3. [補充／官方：KVM API](https://docs.kernel.org/virt/kvm/api.html)：對照 General description、Restrictions、KVM_GET_API_VERSION、KVM_CREATE_VM、KVM_CREATE_VCPU；不逐項讀全部 ioctl。

**自我檢核（4 項）**

- [ ] 1. 固定 vm_init 中的 /dev/kvm fd、VM fd 與 vCPU fd 各代表什麼物件，由哪個 open／ioctl 取得？
- [ ] 2. KVM_CREATE_VM 與 KVM_CREATE_VCPU 各應送到哪個 fd，為什麼這三層 fd 不能互換？
- [ ] 3. KVM_GET_API_VERSION 的用途與文件規定的檢查方式是什麼，API version 與個別 capability 檢查是否處理同一件事？
- [ ] 4. 短練習：沿固定 vm_init 列出最多 8 個建立步驟，每個 ioctl 旁標註輸入 fd 類型與回傳物件；暫時略過 arch 與裝置模擬細節，不必執行。

**選做實驗**：確認 kvm-host 的建置先備，或寫 KVM API version 查詢程式；guest 建置另留時間。

### 第 34 週｜虛擬機記憶體

**必讀（依序開啟）**

- [ ] 1. [補充／固定範例：vm.c](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/src/vm.c)：接上一週，只追記憶體配置與 region 註冊。
- [ ] 2. [補充／官方：KVM API](https://docs.kernel.org/virt/kvm/api.html)：對照 KVM_SET_USER_MEMORY_REGION。

選讀：[x86 KVM MMU](https://docs.kernel.org/virt/kvm/x86/mmu.html) 的位址轉換概觀；內部 shadow MMU／TDP 實作不作為本週門檻。

**自我檢核（4 項）**

- [ ] 1. vm_init 的 mmap 所取得的是哪一種位址，KVM_SET_USER_MEMORY_REGION 如何把它與 guest physical memory 連起來？
- [ ] 2. region 的 slot、guest_phys_addr、memory_size 與 userspace_addr 各描述什麼，哪個是 guest 位址、哪個是 host userspace 位址？
- [ ] 3. guest virtual、guest physical、host virtual 與 host physical 的角色有何不同，host virtual 連續是否保證 host physical 連續？
- [ ] 4. 短練習：為固定範例的 region 畫一張地址圖，並選一個位於 RAM_BASE 範圍內的 guest physical offset，標出它對應的 host buffer offset；只算位移，不必取得真實地址。

**選做實驗**：改變一個範例的記憶體參數並觀察；x86 MMU 內部實作選讀。

### 第 35 週｜vCPU 執行與 VM exit

**必讀（依序開啟）**

- [ ] 1. [補充／固定範例：vm.c](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/src/vm.c)：追 vm_run 中查詢映射大小、mmap(vCPU fd)、KVM_RUN、exit_reason，以及 vm_exit 清理；區分它與 vm_init 中 guest RAM 的 mmap。
- [ ] 2. [補充／官方：KVM API](https://docs.kernel.org/virt/kvm/api.html)：只對照 KVM_GET_VCPU_MMAP_SIZE、KVM_RUN 與 kvm_run 結構。

選讀：[固定版本的 x86 vm.c](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/src/arch/x86/vm.c)：暫存器與啟動狀態；需要時再查 KVM_SET_REGS／KVM_SET_SREGS。

**自我檢核（4 項）**

- [ ] 1. vm_run 為什麼先查 KVM_GET_VCPU_MMAP_SIZE，再 mmap vCPU fd，這個映射與第 34 週 guest RAM 的 mmap 有何不同？
- [ ] 2. KVM_RUN 的 ioctl 回傳值與 kvm_run.exit_reason 分別提供什麼資訊，為什麼成功回傳不代表 guest 程式已全部執行完？
- [ ] 3. 固定 vm_run 如何區分 KVM_EXIT_IO、KVM_EXIT_MMIO 與結束路徑，kvm_run 映射與 VM 資源分別在哪裡清理？
- [ ] 4. 短練習：沿固定 vm_run 畫出『查大小 → mmap → KVM_RUN → 檢查回傳 → 分派 exit_reason → 再執行或結束』，選一個 IO／MMIO 分支標出被呼叫的 handler；不必實跑。

**選做實驗**：在可用環境執行固定範例並記錄一次 VM exit；完整 VMM／裝置模擬選做。

### 第 36 週｜虛擬化小實驗收尾

**必讀（依序開啟）**

- [ ] 1. [補充／固定範例：main.c](https://github.com/sysprog21/kvm-host/blob/75962e148b3e60c75290dfe7b64bd1d54354cb87/src/main.c)：把前面讀過的初始化、執行、清理連成主流程。
- [ ] 2. [補充／官方：KVM API](https://docs.kernel.org/virt/kvm/api.html)：回查範例的 exit reason 與錯誤。

選讀：[QEMU EDU](https://www.qemu.org/docs/master/specs/edu.html)，對照 QEMU 的裝置模擬與 guest driver；EDU 並非 kvm-host 本身提供的裝置。

**自我檢核（4 項）**

- [ ] 1. 固定 main.c 依什麼順序建立 VM、載入 guest、完成啟動設定、執行與清理，initrd／disk 載入由什麼條件決定？
- [ ] 2. guest 執行、I/O／MMIO exit 的處理與使用者端資源清理，分別是 KVM 與這個 VMM 的哪一方責任？
- [ ] 3. main.c 如何接收 vm_run 的結果，正常 VM exit、KVM_RUN 的錯誤與 VMM 程序的退出狀態是否都被以相同方式處理？
- [ ] 4. 短練習：用 main.c、vm.c 整理一張最多 12 個節點的全流程圖，加入一條初始化失敗與一條正常退出路徑；在尚未追讀的裝置細節旁作註記，不必寫新 VMM。

**選做實驗**：完成一次 kvm-host 執行或小修改，保留環境、錯誤與結果；不具備 KVM 時以程式追蹤完成。

## 第 37–42 週：即時 Linux

### 第 37 週｜一般與即時排程政策

**必讀（依序開啟）**

- [ ] 1. [原課程：排程器教材](https://hackmd.io/@sysprog/linux-scheduler)：本輪先看排程類別與一般／即時任務的差異。
- [ ] 2. [原課程：即時排程實務](https://eci.intel.com/docs/3.2/development/performance/rt_scheduling.html)：提前第 40 週的政策入門，先看 SCHED_OTHER、SCHED_FIFO、SCHED_RR、priority、blocking；其他政策選讀。
- [ ] 3. [補充／官方：PREEMPT_RT 理論](https://docs.kernel.org/core-api/real-time/theory.html)：只看一般與即時排程目標，作為下一週搶佔與優先權繼承的入口。

選讀：[EEVDF 理論](https://hackmd.io/@sysprog/HyFtDjMfxg) → [官方摘要](https://docs.kernel.org/scheduler/sched-eevdf.html)。這是公平排程深入內容，讀懂 eligibility、lag、virtual deadline 即可，並非 PREEMPT_RT 的必要前置。

**自我檢核（4 項）**

- [ ] 1. SCHED_OTHER 與即時排程政策各自想滿足什麼需求，公平性與及時執行的目標有何不同？
- [ ] 2. SCHED_FIFO 與 SCHED_RR 遇到相同優先權的可執行任務時，選擇下一個任務的方式有何差別？
- [ ] 3. 即時任務的優先權、被搶佔與因等待資源而阻塞分別代表什麼，為何需要分開討論？
- [ ] 4. 短練習：用 15–30 分鐘畫出兩個相同優先權任務與一個較高優先權任務的示意時間線，分別標出 FIFO、RR 下的執行、阻塞與喚醒；不需修改系統排程政策。

**選做實驗**：觀察不同政策的程序；EEVDF 深入閱讀可在主線完成後再補。

### 第 38 週｜搶佔與優先權反轉

**必讀（依序開啟）**

- [ ] 1. [核心搶佔](https://hackmd.io/@sysprog/linux-preempt)。
- [ ] 2. [Mutex 教材](https://hackmd.io/@sysprog/concurrency-mutex)：這次看 contention 與優先權相關討論。
- [ ] 3. [補充／官方：即時搶佔入口](https://docs.kernel.org/core-api/real-time/index.html)：先讀 Theory of operation 相關內容。

**自我檢核（4 項）**

- [ ] 1. 搶佔與阻塞有何差別，任務無法繼續執行時應如何判斷是哪一種情況？
- [ ] 2. 低優先權任務持有高優先權任務需要的 mutex 時，中優先權任務如何造成優先權反轉？
- [ ] 3. 優先權繼承如何改變持鎖者的排程優先權，它與單純提高等待者的優先權有何不同？
- [ ] 4. 短練習：用 15–30 分鐘畫出高、中、低優先權三個任務與一把 mutex 的時間線，對照有無優先權繼承時的等待路徑；以紙筆分析即可。

**選做實驗**：用簡單執行緒例子重現優先權反轉；避免同時自寫完整鎖實作。

### 第 39 週｜PREEMPT_RT

**必讀（依序開啟）**

- [ ] 1. [PREEMPT_RT 教材](https://hackmd.io/@sysprog/preempt-rt)。
- [ ] 2. [補充／官方：即時搶佔文件](https://docs.kernel.org/core-api/real-time/index.html)：讀 locking 與 interrupts 的說明。

**自我檢核（4 項）**

- [ ] 1. PREEMPT_RT 如何改變核心中可以被搶佔的區段，與一般核心的主要差別是什麼？
- [ ] 2. 在 PREEMPT_RT 下，spinlock_t 與 raw_spinlock_t 的行為與適用情境為何需要區分？
- [ ] 3. 將中斷處理移到執行緒脈絡有什麼作用，哪些中斷處理或特殊脈絡仍需個別確認？
- [ ] 4. 短練習：用 15–30 分鐘從本週文件整理一般核心與 PREEMPT_RT 的鎖、中斷與搶佔對照表，各列一項仍須確認的限制；不需建置 RT 核心。

**選做實驗**：檢查或建置一個 RT 核心，記錄 PREEMPT 組態；建置可延長。

### 第 40 週｜時間與週期任務

**必讀（依序開啟）**

- [ ] 1. [Timer 教材](https://hackmd.io/@sysprog/linux-timer)：這次讀高精度時間與 timeout 的部分。
- [ ] 2. [LKMPG：時間機制](https://sysprog21.github.io/lkmpg/#timekeeping-and-deferred-execution)：對照實作。
- [ ] 3. [原課程：即時排程實務](https://eci.intel.com/docs/3.2/development/performance/rt_scheduling.html)：接第 37 週，本次只看 affinity 與執行設定，不重讀政策入門。

**自我檢核（4 項）**

- [ ] 1. 週期、timeout 與高精度 timer 各自描述什麼，選擇時間機制時需要確認哪些條件？
- [ ] 2. 定時器到期、任務實際開始執行與 deadline 分別代表什麼，如何用它們描述喚醒延遲與 deadline miss？
- [ ] 3. CPU affinity 如何影響週期任務的執行條件，量測時為何還需要記錄時鐘與排程設定？
- [ ] 4. 短練習：用 15–30 分鐘畫出一個週期任務的五次預定喚醒與實際執行時間，標出延遲與期限，再列出量測需要保存的 clock、policy、priority 與 affinity。

**選做實驗**：量測簡單週期任務的喚醒延遲，記錄 clock 與 affinity 設定。

### 第 41 週｜負載與延遲量測

**必讀（依序開啟）**

- [ ] 1. [補充／工具手冊：cyclictest](https://manpages.debian.org/testing/rt-tests/cyclictest.8.en.html)：讀 SYNOPSIS 與 OPTIONS 的 -t、-i、-p、--policy、-D、-m、-q、affinity；手冊是 Debian testing 版本，操作前用 Ubuntu 中的 cyclictest --help 核對差異。
- [ ] 2. [原課程：即時排程實務](https://eci.intel.com/docs/3.2/development/performance/rt_scheduling.html)：只回查 policy、priority、affinity 設定；cyclictest 操作以上方工具手冊為準。

**自我檢核（4 項）**

- [ ] 1. cyclictest 量測的是什麼，這個數值與應用程式完成工作的總時間有何不同？
- [ ] 2. -t、-i、-p、--policy、-D、-m 與 -q 分別控制哪些條件，affinity 又控制什麼？
- [ ] 3. 比較空載與負載時，哪些設定應保持一致，哪些環境與版本資訊必須記錄？
- [ ] 4. 短練習：用 15–30 分鐘依工具手冊草擬一組有限時間的量測命令並逐一註解選項，再列出空載與負載共用的設定；若尚未安裝 cyclictest，先只做命令與紀錄表分析。

**選做實驗**：在安裝的 rt-tests 版本核對選項後，做有限時間的空載／負載量測。

### 第 42 週｜即時階段整理

**必讀（依序開啟）**

- [ ] 1. [PREEMPT_RT](https://hackmd.io/@sysprog/preempt-rt)：回查你測到的現象。
- [ ] 2. [即時排程實務](https://eci.intel.com/docs/3.2/development/performance/rt_scheduling.html)：回查 priority、affinity 與負載設定。

**自我檢核（4 項）**

- [ ] 1. 如何整理測試環境、排程設定、負載與量測方法，讓另一個人能理解比較的條件？
- [ ] 2. 平均延遲、最大觀察延遲與最壞情況保證各自代表什麼，有限次量測能支持哪些結論？
- [ ] 3. 在 VM 中量測即時行為時，host 負載與虛擬化條件會使結果有哪些解讀限制？
- [ ] 4. 短練習：用 15–30 分鐘整理一頁比較報告模板，包含環境、方法、結果與限制；沒有實測數據時使用明確標示的示意數值，並列出仍需驗證的項目。

**選做實驗**：條件允許時比較一般與 RT 核心；VM 結果一併記錄 host 負載與虛擬化條件。

## 第 43–50 週：Rust 核心開發

語言階段使用原課程列出的繁體中文 Rust Book。核心 API 與工具鏈以當週選定的核心版本為準；先追小型範例，完整驅動實作另作延伸。

### 第 43 週｜Rust 與 Cargo 入門

**必讀（依序開啟）**

- [ ] 1. [Rust Book：起步](https://rust-lang.tw/book-tw/ch01-00-getting-started.html)：安裝、Hello World、Cargo。
- [ ] 2. [Rust Book：一般程式觀念](https://rust-lang.tw/book-tw/ch03-00-common-programming-concepts.html)：變數、型別、函式、流程控制；用 C 經驗快速對照。

**自我檢核（4 項）**

- [ ] 1. Cargo.toml、原始碼與 Cargo 的建立、建置、執行、檢查指令各自在 Rust 專案中負責什麼？
- [ ] 2. Rust 的變數綁定、mut 與基本型別如何使用，與你熟悉的 C 寫法有哪些差別？
- [ ] 3. Rust 函式與 if 等流程控制中的運算式如何產生值，分號對結果有什麼影響？
- [ ] 4. 短練習：用 15–30 分鐘閱讀教材的一段基本 Rust 程式，標出變數型別、可變性、函式參數與回傳值，再手改一個輸入或分支並預測結果；不必先安裝工具鏈。

**選做實驗**：用 Cargo 編譯與測試一個小型使用者程式。

### 第 44 週｜ownership 與 borrowing

**必讀（依序開啟）**

- [ ] 1. [Ownership](https://rust-lang.tw/book-tw/ch04-01-what-is-ownership.html)。
- [ ] 2. [References 與 borrowing](https://rust-lang.tw/book-tw/ch04-02-references-and-borrowing.html)。
- [ ] 3. [Slice](https://rust-lang.tw/book-tw/ch04-03-slices.html)。

**自我檢核（4 項）**

- [ ] 1. move 發生後原變數還能做什麼，Rust 如何用所有權與作用域管理物件生命週期？
- [ ] 2. shared borrow 與 mutable borrow 各允許哪些操作，同一時間使用它們時有哪些限制？
- [ ] 3. slice 如何關聯到原始資料，它的有效期限為何需要與資料的生命週期一起考慮？
- [ ] 4. 短練習：用 15–30 分鐘挑教材中的 move、shared borrow 與 mutable borrow 各一段例子，畫出變數與物件的關係，標出哪些行可合法使用原資料；不需編譯。

**選做實驗**：把簡單 C 字串／陣列操作改寫成 Rust，觀察借用錯誤。

### 第 45 週｜結構、enum 與錯誤處理

**必讀（依序開啟）**

- [ ] 1. [Struct](https://rust-lang.tw/book-tw/ch05-00-structs.html)。
- [ ] 2. [Enum 與 match](https://rust-lang.tw/book-tw/ch06-00-enums.html)。
- [ ] 3. [錯誤處理](https://rust-lang.tw/book-tw/ch09-00-error-handling.html)：重點看 Result、Option 與 ?。

**自我檢核（4 項）**

- [ ] 1. struct 與 enum 適合表示哪些不同種類的資料，match 如何處理各種 enum 狀態？
- [ ] 2. Option 與 Result 分別表示什麼，什麼情況應用沒有值與操作失敗來描述？
- [ ] 3. ? 如何影響函式的成功與錯誤路徑，它對所在函式的回傳型別有哪些要求？
- [ ] 4. 短練習：用 15–30 分鐘從教材挑一段使用 Result 與 ? 的函式，分別畫出成功與失敗的控制流程，再把其中一個 ? 展開成 match 的草稿；不需建置專案。

**選做實驗**：將一個 C 小範例改成 Rust，處理成功與失敗。

### 第 46 週｜trait、lifetime 與共享所有權

**必讀（依序開啟）**

- [ ] 1. [泛型入門](https://rust-lang.tw/book-tw/ch10-01-syntax.html)：只看型別參數與簡單函式／結構範例。
- [ ] 2. [Trait](https://rust-lang.tw/book-tw/ch10-02-traits.html)：只看基本宣告、實作與 bounds。
- [ ] 3. [Lifetime](https://rust-lang.tw/book-tw/ch10-03-lifetime-syntax.html)：只看借用期限與簡單標註。
- [ ] 4. [智慧指標入口](https://rust-lang.tw/book-tw/ch15-00-smart-pointers.html)：只讀 Box 與 Drop；引用計數、內部可變性列選讀，這些使用者空間型別不等於核心 API。

選讀：同章 Rc、RefCell 的概念；先不要求掌握完整型別組合。

**自我檢核（4 項）**

- [ ] 1. 泛型的型別參數與 trait bound 各自表達什麼，trait 宣告與實作如何連接介面與行為？
- [ ] 2. lifetime 標註描述借用之間的什麼關係，它與物件實際配置或釋放的時機有何不同？
- [ ] 3. Box 與 Drop 分別負責什麼，如何沿著作用域判斷資源清理的時機？
- [ ] 4. 短練習：用 15–30 分鐘各挑一個簡單 trait、lifetime 與 Box／Drop 例子，標出型別要求、借用關係與清理位置；先不加入 Rc 或 RefCell。

**選做實驗**：各寫一個最小 trait／借用範例；Rc／RefCell／完整智慧指標組合可延後。

### 第 47 週｜Rust 核心環境與研究入口

**必讀（依序開啟）**

- [ ] 1. [原課程：Rust for Linux 研究](https://hackmd.io/@hank20010209/SJmVNPMQ2)：先看研究背景與架構。
- [ ] 2. [補充／官方：核心 Rust 概觀](https://docs.kernel.org/rust/general-information.html)：先讀 no_std、abstractions 與 bindings，釐清與使用者空間 Rust／Cargo 的差異。
- [ ] 3. [補充／官方：Rust 核心 Quick Start](https://docs.kernel.org/rust/quick-start.html)：工具鏈、rustavailable、組態與建置；以選定核心版本為準。

環境範圍：現有 Ubuntu 24.04 可作建置端；官方 Quick Start 另有 Ubuntu 24.04 的版本套件說明，先按目標核心核對 rustc、rust-src、bindgen 與 LLVM，不直接假設第 43 週 Cargo 的預設工具鏈可用。

**自我檢核（4 項）**

- [ ] 1. 核心 Rust 的 no_std 代表什麼，為何不能直接把使用者空間 Cargo 程式的依賴與 API 搬進核心？
- [ ] 2. kernel abstractions 與 C bindings 各負責什麼，驅動程式使用這些介面時應如何理解安全責任？
- [ ] 3. 目標核心版本、rustc、rust-src、bindgen、LLVM 與 rustavailable 的檢查有什麼關係？
- [ ] 4. 短練習：用 15–30 分鐘閱讀 Quick Start 的 Ubuntu 24.04 說明，列出目標核心所需工具與檢查步驟，再畫出 Rust 模組、抽象與 bindings 的關係；只整理清單，不需安裝或建置。

**選做實驗**：固定核心／工具鏈，執行 rustavailable，完成 CONFIG_RUST 與 sample 建置；環境較重可延長。

### 第 48 週｜第一個 Rust 核心範例

**必讀（依序開啟）**

- [ ] 1. [原課程：Rust 核心模組專題](https://hackmd.io/@sysprog/Sk8IMQ9S2)：看環境與最小範例的說明。
- [ ] 2. [補充／官方：rust_minimal 範例](https://github.com/torvalds/linux/blob/master/samples/rust/rust_minimal.rs)：固定從這個範例開始，切換至自己核心的相同 tag；只讀／修改 safe API 下的輸出與簡單狀態，unsafe 深入留到第 49 週。
- [ ] 3. [補充／官方：Quick Start](https://docs.kernel.org/rust/quick-start.html)：回查建置。

**自我檢核（4 項）**

- [ ] 1. rust_minimal 中的 module!、模組型別與 init 如何連接模組資訊與初始化流程？
- [ ] 2. 範例的狀態如何建立並保存，Drop 如何對應到模組清理與 C 模組的退出流程？
- [ ] 3. 為何閱讀範例與 Quick Start 時要對齊同一核心版本，API 差異可能出現在哪些位置？
- [ ] 4. 短練習：用 15–30 分鐘在 rust_minimal 的閱讀副本或筆記中標出 module!、init、狀態、錯誤回傳與 Drop，畫出生命週期；可只草擬一行輸出修改，不需建置核心。

**選做實驗**：改一個輸出或小型狀態，建置／載入，對照 C Hello 模組。

### 第 49 週｜unsafe 邊界與資源管理

**必讀（依序開啟）**

- [ ] 1. [Rust Book：Unsafe Rust](https://rust-lang.tw/book-tw/ch19-01-unsafe-rust.html)：只看 unsafe 的權限與需要維持的條件；這不是略過所有檢查的開關。
- [ ] 2. [原課程：Rust 模組專題](https://hackmd.io/@sysprog/Sk8IMQ9S2)：看抽象、資源與範例限制。
- [ ] 3. [補充／官方：Rust 範例](https://github.com/torvalds/linux/tree/master/samples/rust)：追蹤所有權、錯誤回傳與清理。

**自我檢核（4 項）**

- [ ] 1. unsafe 允許哪些額外操作，哪些型別、借用與安全條件仍然需要成立？
- [ ] 2. safe abstraction、unsafe 實作與 C bindings 的責任如何區分，安全介面的保證依賴什麼？
- [ ] 3. 閱讀涉及指標、所有權或釋放的 unsafe 程式碼時，需要確認哪些有效性與生命週期條件？
- [ ] 4. 短練習：用 15–30 分鐘挑 Rust Book 的一段 unsafe 例子，逐行標出額外操作、所需條件與由誰維持條件，再對照核心文件的抽象與 bindings 分工；不新增 unsafe 功能。

**選做實驗**：對所選範例作簡單註解；unsafe 區段以閱讀為主，不要求新增 unsafe 功能。

### 第 50 週｜Rust 小專題收尾

**必讀（依序開啟）**

- [ ] 1. [Rust 模組教材](https://hackmd.io/@sysprog/Sk8IMQ9S2)：回查本週實作需要的部分。
- [ ] 2. [官方 rust_minimal 範例](https://github.com/torvalds/linux/blob/master/samples/rust/rust_minimal.rs)：回查與 C 模組對應的初始化、狀態與清理流程，沿用第 48 週選定的核心版本。

**自我檢核（4 項）**

- [ ] 1. rust_minimal 與 C Hello 模組的初始化、狀態保存與退出介面如何相互對照？
- [ ] 2. 兩個最小範例中，哪些初始化步驟可能失敗，已建立的狀態如何清理？沒有取得額外資源的範例與實際驅動有何差別？
- [ ] 3. 評估一個 Rust 模組的小修改時，如何界定行為、影響範圍與需要驗證的項目？
- [ ] 4. 短練習：用 15–30 分鐘對照第 3 週 C Hello 與同版本 rust_minimal，整理初始化、狀態、錯誤與清理四欄表，再草擬一個小修改的預期行為與驗證方式；不需實際載入。

**選做實驗**：完成並驗證一個有界的小修改；所選版本支援時再加簡單裝置功能。

## 第 51–54 週：核心開發與貢獻

### 第 51 週｜選題與重現

**必讀（依序開啟）**

- [ ] 1. [原課程：第一次 patch 經驗](https://hackmd.io/@rhythm/BkjJeugOv)：先理解完整流程。
- [ ] 2. [補充／官方：核心 coding style](https://docs.kernel.org/process/coding-style.html)：先讀與選定修改有關的規則。

選讀：[LKL 教材](https://hackmd.io/@sysprog/linux-lkl)。想研究在使用者空間重用核心時再看；它不是準備 patch 的必要前置。

**自我檢核（4 項）**

- [ ] 1. 什麼樣的小問題適合作為第一份 patch 的題目，如何界定目標專案與修改範圍？
- [ ] 2. 基準版本、操作步驟、預期行為與實際行為如何組成可供他人理解的重現紀錄？
- [ ] 3. coding style 在修改中有什麼用途，為何應聚焦與本次修改相關的規則？
- [ ] 4. 短練習：用 15–30 分鐘從已有筆記或教材挑一個小問題，寫出專案、版本、預期與實際行為、重現步驟及相關風格規則；尚未實測的部分明確標示。

**選做實驗**：從前面紀錄挑一個小問題並重現；LKL 不作為選題或 patch 的先決條件。

### 第 52 週｜修改與驗證

**必讀（依序開啟）**

- [ ] 1. [補充／官方：提交 patch 指南](https://docs.kernel.org/process/submitting-patches.html)：讀修改範圍、說明問題與測試相關內容。
- [ ] 2. [補充／官方：coding style](https://docs.kernel.org/process/coding-style.html)：套用到自己的修改。

**自我檢核（4 項）**

- [ ] 1. 如何判斷一份修改是否只有單一目的，哪些整理或修正應另外處理？
- [ ] 2. 修改前後應使用什麼驗證方式，如何確認結果與想解決的問題直接相關？
- [ ] 3. 說明修改時，問題、改動原因與驗證結果應如何連接，哪些限制需要如實記錄？
- [ ] 4. 短練習：用 15–30 分鐘針對上一週的小問題草擬最小修改與驗證表，列出修改前後的預期差異、使用版本及尚未完成的檢查；無須先編譯核心。

**選做實驗**：完成一個小修改，保存前後結果與版本。

### 第 53 週｜commit 與 patch 草稿

**必讀（依序開啟）**

- [ ] 1. [補充／官方：提交 patch 指南](https://docs.kernel.org/process/submitting-patches.html)：commit message、Signed-off-by、format-patch、maintainer 等流程。
- [ ] 2. [補充／官方：Posting patches](https://docs.kernel.org/process/5.Posting.html)：patch 準備與拆分。

**自我檢核（4 項）**

- [ ] 1. commit message 的標題與本文各應交代什麼，如何讓未看過你的筆記的人理解改動原因？
- [ ] 2. Signed-off-by 的用途與責任是什麼，它與測試結果或審查認可有何不同？
- [ ] 3. format-patch、patch 拆分與維護者查找各處理什麼，準備寄送前需要確認哪些專案流程？
- [ ] 4. 短練習：用 15–30 分鐘為前面的假設修改草擬標題、問題、理由與驗證說明，並依文件列出查找維護者的步驟；不宣稱未做的測試，也不需寄送。

**選做實驗**：產生 patch 草稿，核對目標專案與維護者；寄送不是本週完成門檻。

### 第 54 週｜自我審查與學習回顧

**必讀（依序開啟）**

- [ ] 1. [補充／官方：Posting patches](https://docs.kernel.org/process/5.Posting.html)：回查審查與版本修訂方式。
- [ ] 2. [原課程：提交第一份 patch 經驗](https://hackmd.io/@steven1lung/submitting-patches)：對照實務紀錄。

**自我檢核（4 項）**

- [ ] 1. 自我審查一份 patch 時，如何確認問題、改動範圍與驗證證據互相對應？
- [ ] 2. 收到審查意見後，如何整理修訂、版本差異與回應，讓下一輪容易比較？
- [ ] 3. 回顧學習成果時，如何區分已能解釋的機制、已完成的實驗與仍待驗證的理解？
- [ ] 4. 短練習：用 15–30 分鐘審查自己的 patch 草稿或教材案例，列出三個需要確認的問題，再整理已理解、已實測與待補課的簡表；不需提交或啟動新環境。

**選做實驗**：整理可供審查的 patch 與重現步驟，回顧 54 個單元；實際寄送另行安排。

## 需要時再補的教材

以下是回查入口，不必在開始前全部讀完。

| 遇到的問題 | 補充連結 | 建議位置 |
|---|---|---|
| 整數型別、溢位、符號轉換 | [數值系統](https://hackmd.io/@sysprog/c-numerics) | 第 2 週或遇到型別問題時 |
| 清理與 goto | [流程控制](https://hackmd.io/@sysprog/c-control-flow) | 第 9、24、30 週 |
| 函式指標與核心物件介面 | [C 物件導向寫法](https://hackmd.io/@sysprog/c-oop) | 第 13、18 週 |
| 編譯器與最佳化 | [編譯器原理](https://hackmd.io/@sysprog/c-compiler-construction)、[最佳化](https://hackmd.io/@sysprog/c-compiler-optimization) | 第 4 週後 |
| 動態函式庫與載入 | [動態連結](https://hackmd.io/@sysprog/c-dynamic-linkage) | 第 4–5 週後 |
| 非連續資料的組織 | [linked list](https://hackmd.io/@sysprog/c-linked-list) | 第 6、13 週 |
| 排程器或查找結構 | [紅黑樹](https://hackmd.io/@sysprog/linux-rbtree)、[hash table](https://hackmd.io/@sysprog/linux-hashtable) | 第 6、20、37 週 |
| 小物件配置器 | [slab 家族教材](https://hackmd.io/@sysprog/linux-slab) | 第 9 週後；對照實驗版本 |
| 檔案 I/O 的使用者介面 | [Stream I/O](https://hackmd.io/@sysprog/c-stream-io)、[CS:APP 第 10 章](https://hackmd.io/@sysprog/CSAPP-ch10) | 第 13、20 週 |
| 讀多寫少的同步 | [RCU](https://hackmd.io/@sysprog/linux-rcu)、[LKMPG RCU](https://sysprog21.github.io/lkmpg/#readcopyupdate-rcu) | 第 29 週後 |
| cache coherence 與鎖擴展性 | [spinlock scalability](https://hackmd.io/@sysprog/linux-spinlock-scalability) | 第 29 週後 |
| 硬體排序與 barrier | [Memory Barrier 投影片](https://elinux.org/images/a/ab/Bueso.pdf) | 第 29 週後 |
| 更完整的 cache 背景 | [每位程式開發者都該有的記憶體知識](https://sysprog21.github.io/cpumemory-zhtw/) | 第 15 週後；選讀 |

## 每週筆記模板

可複製到自己的實驗筆記中。教材與自我檢核進度直接勾選各週的清單。

```text
週次／主題：
開始與完成日期：
核心／工具／範例版本：
本週讀到的範圍：
問題 1 回答／教材依據：
問題 2 回答／教材依據：
問題 3 回答／教材依據：
短練習結果（流程圖、程式註記或小修改）：
尚未理解、需要回查的地方：
選做實驗命令與步驟（有做才填）：
預期與實際結果：
容易記錯的觀念：
下週需要回查的連結：
程式碼 commit／紀錄位置：
```

## 連結維護

本課表以 2026-10-05 的教材入口整理與修訂。LKMPG 使用章節名稱的連結，Rust Book 使用章節頁面；若網站改版造成定位失效，可先回到該教材目錄找同名章節。

原課程的 HackMD、SlideShare 等外部服務可能暫時限制存取；可回到 [原課程頁](https://wiki.csie.ncku.edu.tw/linux/schedule) 找對應教材與錄影。官方文件通常跟隨開發版本更新，操作時應對照自己選定的版本。

# 概念與原理

## 什麼是 C++？

C++ 是一種靜態型別、編譯型的通用程式語言。

由 Bjarne Stroustrup 於 1979 年在貝爾實驗室以 C 語言為基礎擴充開發而成。

與 Python 等直譯型語言不同，C++ 在執行前必須先經過**編譯器（Compiler）**將程式碼完整轉譯成電腦 CPU 能夠直接執行的**機器二進位碼（Machine Code）**。

因此，C++ 具有極高的執行速度與極低的記憶體開銷，廣泛應用於作業系統、遊戲引擎、高效能伺服器、自動駕駛與嵌入式系統。

---

## 第一個 C++ 程式：Hello World

以下是標準的 C++ 程式骨架：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

### 程式碼逐行拆解說明

1. **`#include <iostream>`**：
   
   引入標準輸入輸出串流標頭檔（Input/Output Stream）。
   
   告訴編譯器我們要使用終端機印出文字與讀取鍵盤輸入的功能。

2. **`int main()`**：
   
   主函式，**整個程式執行的唯一入口起點**。
   
   所有 C++ 程式都必須從 `main()` 的第一行開始執行。

3. **`std::cout << "Hello, World!" << std::endl;`**：
   
   - `std::cout`：代表標準輸出（Standard Output，即終端機螢幕）。
   - `<<`：串流插入運算子，將右側的資料「送進」輸出串流中。
   - `"Hello, World!"`：要輸出的字串內容。
   - `std::endl`：輸出換行並清空緩衝區（相當於 `\n` 並立即刷新）。
   - `;`：分號，代表這行指令結束（C++ 每行語句結尾必須加上分號）。

4. **`return 0;`**：
   
   向作業系統回傳狀態碼 `0`，代表程式正常執行完畢並退出。

---

## 命名規則與風格慣例

變數、函式與類別的命名必須符合以下規定：

1. 只能由**英文字母（大小寫敏感）**、**數字**與**底線（_）**組成。
2. **數字絕對不能作為開頭**（例如 `2users` 是非法的，`user2` 是合法的）。
3. 不能與 C++ 的**保留關鍵字**撞名（例如 `int`、`class`、`return`）。

### 社群常用命名風格

| 元素類別 | 命名風格 | 舉例 | 說明 |
| :--- | :--- | :--- | :--- |
| **變數名稱** | 蛇形命名 (snake_case) | `student_age`, `total_score` | 全部小寫，以底線連接 |
| **常數名稱** | 全大寫蛇形 (UPPER_SNAKE) | `MAX_BUFFER_SIZE`, `PI` | 全部大寫，代表不可變數值 |
| **函式名稱** | 蛇形 或 小駝峰 (camelCase) | `calculate_total()`, `getUserName()` | 動詞開頭，表達行為 |
| **類別名稱** | 帕斯卡命名 (PascalCase) | `StudentManager`, `HttpServer` | 大駝峰，每個單詞首字母大寫 |

---

# 核心語法與關鍵字字典對照表

| 語法 / 關鍵字 / 函式名稱 | 類型 | 主要用途 | 典型適用情境 |
| :--- | :--- | :--- | :--- |
| **[[#main() 程式入口主函數\|main()]]** | 核心函式 | **整個 C++ 應用程式的唯一執行入口起點** | 程式啟動第一站與回傳狀態碼規範 |
| **[[#std::cout 標準輸出\|std::cout]]** | 標準串流 | **終端機標準輸出串流 (搭配 `<<`)** | 向終端機螢幕印出文字、數值與訊息 |
| **[[#std::cin 標準輸入\|std::cin]]** | 標準串流 | **鍵盤標準輸入串流 (搭配 `>>`)** | 讀取使用者從鍵盤鍵入的資料 |
| **[[#std::getline() 讀取整行字串 (含空格文字)\|std::getline()]]** | 標準函式 | **讀取包含空格的整行文字** | 讀取完整英文姓名、地址或一整行句子 |
| **[[#常數 const\|const]]** | 修飾詞 | **宣告唯讀常數，防範資料被意外修改** | 定義數學常數、固定配置參數與常數傳參 |
| **[[#型別轉換 (Type Casting)\|static_cast<T>()]]** | 轉型運算子 | **編譯期安全的顯式型別轉換** | 浮點數轉整數、整數除法前精準轉型 |
| **[[#條件判斷 (if / else)\|if / else]]** | 控制結構 | **依條件真假決定程式執行分支** | 條件過濾、邏輯決策與防呆判斷 |
| **[[#switch-case 多分支選擇\|switch-case]]** | 控制結構 | **針對單一整數/字元變數進行多分支等值比對** | 狀態機切換、選單指令選擇 |
| **[[#迴圈結構\|for]]** | 迴圈結構 | **重複執行固定次數的程式碼區塊** | 計數器迭代、依索引遍歷陣列 |
| **[[#迴圈結構\|while]]** | 迴圈結構 | **當條件滿足時重複執行程式碼** | 條件未滿足前持續等待或執行 |
| **[[#2.1 std::string 標準字串\|std::string]]** | STL 類別 | **動態長度字串容器** | 儲存文字、字串串接、長度計算與子字串 |
| **[[#2.2 const char* 與 C 風格字串\|const char*]]** | 原生型別 | **指向唯讀常數區的字元指標** | 字串常數、對接 C 語言 API |
| **[[#2.3 原生固定陣列 (Array)\|原生陣列]]** | 核心語法 | **固定長度連續記憶體集合** | 數量固定且追求極致效能的資料集合 |
| **[[#2.4 std::vector 動態陣列向量\|std::vector<T>]]** | STL 容器類別 | **長度可動態自動增長的連續記憶體陣列** | 取代傳統固定陣列的標準陣列首選 |
| **[[#2.5 結構體 struct\|struct]]** | 關鍵字 | **將多個不同型別的資料欄位打包** | 純資料封裝、公開屬性結構體 |
| **[[#2.6 類別 class\|class]]** | 關鍵字 | **物件導向核心：封裝私有屬性與公有方法** | 物件導向程式設計、安全資料管理 |
| **[[#參數傳遞機制：傳值 vs 傳參考\|傳參考 &]]** | 核心機制 | **建立變數別名，直接操作外部原資料** | 高效傳遞參數、讓函式修改外部變數 |
| **[[#指標變數與提領運算子\|指標 *]]** | 核心機制 | **儲存記憶體位址並透過提領 `*` 存取資料** | 動態記憶體操作、底層系統操作 |
| **[[#空指標 nullptr\|nullptr]]** | 常數值 | **型別安全的空指標常數 (取代 NULL)** | 初始化未綁定位址的指標，防呆判斷 |
| **[[#std::sqrt() 計算平方根\|std::sqrt()]]** | 數學函式 | **計算數值的平方根 (開根號)** | 幾何運算、物理計算與距離公式 |
| **[[#std::pow() 計算次方次冪\|std::pow()]]** | 數學函式 | **計算指定底數的指數次方 ($base^{exp}$)** | 冪次計算、複利計算與高階多項式 |
| **[[#std::abs() 計算絕對值\|std::abs()]]** | 數學函式 | **取得數值的非負絕對值** | 計算兩點距離、誤差計算 |
| **[[#std::max() 與 std::min() 取最大值與最小值\|std::max() / std::min()]]** | 演算法函式 | **比較並取得兩數或數值清單之極值** | 邊界限制、極值比對與分數過濾 |
| **[[#std::round() / std::floor() / std::ceil() 數值取整\|round / floor / ceil]]** | 數學函式 | **四捨五入、無條件捨去、無條件進位** | 數值取整、金額計算、分頁計算 |
| **[[#std::to_string() 數值轉為字串\|std::to_string()]]** | 字串函式 | **將整數或浮點數轉換為標準字串** | 格式化文字輸出、數值串接至字串 |
| **[[#std::stoi() 與 std::stod() 字串轉整數與浮點數\|std::stoi() / std::stod()]]** | 字串函式 | **將字串解析為整數或浮點數** | 解析使用者輸入文字、讀取數值檔案 |
| **[[#std::sort() 快速排序演算法\|std::sort()]]** | 演算法函式 | **快速排序容器或陣列 ($O(N \log N)$)** | 成績排序、資料由小到大或由大到小整理 |
| **[[#std::reverse() 元素順序反轉\|std::reverse()]]** | 演算法函式 | **原地前後顛倒容器或字串的元素順序** | 字串反轉、歷史紀錄逆序顯示 |
| **[[#std::find() 線性搜尋元素\|std::find()]]** | 演算法函式 | **線性查找容器中特定數值第一次出現處** | 搜尋資料、確認元素是否存在於清單 |
| **[[#std::count() 統計元素出現次數\|std::count()]]** | 演算法函式 | **統計特定數值在容器中出現的總次數** | 票數統計、字元出現頻率分析 |
| **[[#std::isdigit() 與 std::isalpha() 字元檢查\|isdigit() / isalpha()]]** | 字元函式 | **檢查字元是否為數字或英文字母** | 輸入格式驗證、文字解析與分詞 |
| **[[#std::tolower() 與 std::toupper() 英文字母大小寫轉換\|tolower() / toupper()]]** | 字元函式 | **單一英文字元之大小寫轉換** | 不分大小寫的比對、文字標準化 |
| **[[#rand() 與 srand() 產生隨機數\|rand() / srand()]]** | 隨機函式 | **生成偽隨機整數 (需先以 srand 設種子)** | 遊戲骰子、隨機抽籤與蒙地卡羅模擬 |

---

# 資料型別 (Data Types)

## 什麼是變數？

變數是電腦記憶體中用來儲存資料的「具名容器」。

在 C++ 中，**宣告變數時必須明確指定其資料型別**，一旦宣告後該變數就只能存放該型別的資料。

```cpp
int age = 20;               // 宣告整數變數 age
double price = 99.5;        // 宣告浮點數變數 price
char grade = 'A';           // 宣告字元變數 grade (單引號)
std::string name = "Alice"; // 宣告字串變數 name (雙引號，需 #include <string>)
bool is_active = true;      // 宣告布林變數 is_active
```

---

## 1. 基本資料型別 (Primitive Types)

基本型別是 C++ **語言編譯器原生內建**的最底層資料型別，直接對應電腦 CPU 暫存器與記憶體硬體。

| 基本型別名稱 | 關鍵字 | 佔用記憶體大小 | 代表意義與數值範圍 | 範例 |
| :--- | :--- | :---: | :--- | :--- |
| **整數型** | `int` | 4 位元組 (Bytes) | 儲存整數，範圍約 -21 億 到 +21 億 | `int count = 100;` |
| **長整數** | `long long` | 8 位元組 | 儲存超大整數，範圍約 $-9 \times 10^{18} \sim +9 \times 10^{18}$ | `long long total = 9000000000LL;` |
| **雙精度浮點** | `double` | 8 位元組 | 儲存帶小數點的實數（約 15 位有效數字，**小數首選**） | `double pi = 3.14159265;` |
| **單精度浮點** | `float` | 4 位元組 | 儲存帶小數點的實數（約 6~7 位有效數字，結尾加 `f`） | `float weight = 65.5f;` |
| **字元型** | `char` | 1 位元組 | 儲存單一字元（用**單引號**包覆，如 `'A'`） | `char level = 'S';` |
| **布林型** | `bool` | 1 位元組 | 邏輯真假值：`true` (1) 或 `false` (0) | `bool passed = true;` |
| **無回傳型別** | `void` | 0 位元組 | 用於函式代表不回傳任何資料 | `void print_message();` |

---

## 2. 複合資料型別 (Compound Types)

複合型別是由**一個或多個基本型別組合包裝而成**的高階資料結構（包含標準庫提供的物件與自訂型別）。

**為什麼字串屬於複合型別？**  
因為字串是由**多個連續字元 (`char`)** 組成的集合體，C++ 透過標準庫 `<string>` 封裝了記憶體動態擴展、長度計算與字串串接等高階功能。

| 複合型別名稱 | 關鍵字 / 宣告方式 | 所屬標頭檔 | 代表意義與功能說明 | 範例 |
| :--- | :--- | :--- | :--- | :--- |
| **標準字串** | `std::string` | `<string>` | 可動態調整長度的文字字串容器（**字串首選**） | `std::string name = "Matthew";` |
| **C 風格字串指標** | `const char*` | 內建 | 原生指向唯讀常數區的字元陣列指標 | `const char* msg = "Hello";` |
| **原生固定陣列** | `型別 變數名[長度]` | 內建 | 存放固定長度、同型別資料的連續記憶體空間 | `int scores[5] = {90, 80};` |
| **動態陣列向量** | `std::vector<型別>` | `<vector>` | 長度可隨時自動增長與縮小的現代連續陣列 | `std::vector<int> nums = {1, 2};` |
| **結構體** | `struct` | 內建 | 將多個不同型別的資料打包成一個自訂型別 | `struct Point { int x; int y; };` |
| **類別物件** | `class` | 內建 | 物件導向核心：封裝私有屬性與公有操作方法 | `class Student { ... };` |

> **字元 char 與 字串 string 的引號差別**：  
> - **單引號 `'A'`**：代表**單一字元 (`char`)**，內部只能放 1 個字元，佔 1 個位元組。  
> - **雙引號 `"Hello"`**：代表**字串 (`std::string` 或字串常數)**，可以放多個字元組成的連續文字。

---

### 2.1 std::string 標準字串

- **使用時機**：處理任何文字資料、文章、使用者輸入時的**首選字串型別**。長度可隨意動態增長，提供大量內建字串處理方法。
- **所屬標頭檔**：`#include <string>`

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello";
    
    // 1. 字串串接 (使用 += 運算子)
    str += " World";
    
    // 2. 取得字串長度 (.length() 或 .size())
    std::cout << "字串長度: " << str.length() << "\n"; // 輸出: 11
    
    // 3. 依索引存取單一字元 (從 0 開始)
    std::cout << "第一個字元: " << str[0] << "\n";      // 輸出: H
    
    // 4. 擷取子字串 .substr(起始索引, 長度)
    std::string sub = str.substr(0, 5);
    std::cout << "擷取子字串: " << sub << "\n";        // 輸出: Hello
    
    return 0;
}
```

---

### 2.2 const char* 與 C 風格字串

- **使用時機**：指向字串常數（例如直接寫在程式碼中的文字 `"Hello"`），或需要對接傳統 C 語言函式庫時。
- **特性**：本質是一個**指向唯讀記憶體區段的字元指標**，以結尾的隱藏空字元 `\0` 作為字串結束記號。

```cpp
#include <iostream>

int main() {
    // 指向唯讀常數區的字串
    const char* c_str = "Hello C++";

    std::cout << "輸出 C 風格字串: " << c_str << "\n";
    std::cout << "第一個字元: " << *c_str << "\n"; // 輸出: H
    return 0;
}
```

---

### 2.3 原生固定陣列 (Array)

- **使用時機**：資料數量在編譯期就已**完全固定**（例如一週有 7 天、一年有 12 個月），不需要動態增減長度時使用。
- **特性**：佔用一整塊連續的記憶體，索引從 `0` 開始，最大合法索引為 `長度 - 1`。

```cpp
#include <iostream>

int main() {
    // 宣告並初始化長度為 5 的整數陣列
    int numbers[5] = {10, 20, 30, 40, 50};

    // 透過索引修改數值
    numbers[0] = 99;

    // 傳統 for 迴圈遍歷陣列
    for (int i = 0; i < 5; i++) {
        std::cout << numbers[i] << " ";
    }
    std::cout << "\n"; // 輸出: 99 20 30 40 50
    return 0;
}
```

---

### 2.4 std::vector 動態陣列向量

- **使用時機**：**現代 C++ 陣列容器首選**。當陣列長度無法預先確定、需要隨時新增（`.push_back()`）或刪除（`.pop_back()`）元素時使用。
- **所屬標頭檔**：`#include <vector>`

```cpp
#include <iostream>
#include <vector>

int main() {
    // 1. 宣告整數 vector
    std::vector<int> nums;

    // 2. 尾端動態新增元素
    nums.push_back(100);
    nums.push_back(200);
    nums.push_back(300);

    // 3. 取得目前元素數量 .size()
    std::cout << "目前元素數量: " << nums.size() << "\n"; // 輸出: 3

    // 4. 現代範圍 for 迴圈遍歷 (Range-based for loop)
    for (int n : nums) {
        std::cout << n << " ";
    }
    std::cout << "\n"; // 輸出: 100 200 300

    // 5. 移除最後一個元素
    nums.pop_back();

    return 0;
}
```

---

### 2.5 結構體 struct

- **使用時機**：當需要將多個不同型別的資料欄位（例如座標的 X 和 Y、學生的姓名與成績）**打包整合為單一自訂型別**時使用。
- **特性**：內部的成員預設皆為 **`public`（公開自由存取）**。

```cpp
#include <iostream>
#include <string>

// 定義一個 Student 結構體
struct Student {
    std::string name;
    int age;
    double gpa;
};

int main() {
    // 建立結構體實例並賦值
    Student s1 = {"Alice", 20, 3.85};

    // 透過點號 . 存取內部成員
    std::cout << "學生姓名: " << s1.name 
              << ", 年齡: " << s1.age 
              << ", GPA: " << s1.gpa << "\n";
    return 0;
}
```

---

### 2.6 類別 class

- **使用時機**：**物件導向程式設計 (OOP) 的核心**。當需要將「資料屬性（私有變數）」與「行為邏輯（公有函式）」封裝在一起，對外提供安全的操作介面時使用。
- **特性**：內部的成員預設皆為 **`private`（私有保護，外部不可隨意篡改）**。

```cpp
#include <iostream>
#include <string>

class BankAccount {
private:
    std::string owner_;
    double balance_ = 0.0; // 私有變數：外部無法直接修改存款金額

public:
    // 建構子 (Constructor)
    BankAccount(std::string owner, double initial_balance) 
        : owner_(owner), balance_(initial_balance) {}

    // 公有方法：存款
    void deposit(double amount) {
        if (amount > 0) balance_ += amount;
    }

    // 公有方法：查詢餘額
    double get_balance() const {
        return balance_;
    }
};

int main() {
    BankAccount account("Matthew", 1000.0);
    account.deposit(500.0);
    std::cout << "當前帳戶餘額: $" << account.get_balance() << "\n"; // 輸出: $1500
    return 0;
}
```

---

## 常數 const

當一個變數的值在初始化之後**絕對不允許被修改**時，可以在型別前方加上 `const` 修飾詞。

```cpp
const double PI = 3.14159;
// PI = 3.14; // 編譯錯誤：不可修改 const 變數的值
```

> **注意事項**：`const` 變數在宣告時**必須立即初始化賦值**，否則會產生編譯錯誤。

---

## 型別轉換 (Type Casting)

將一種資料型別的值轉換為另一種型別。

### 1. 隱式型別轉換 (自動轉換)

當不同型別進行運算時，編譯器會自動將小範圍型別提升為大範圍型別：

```cpp
int a = 5;
double b = 2.0;
double result = a / b; // a 會自動提升為 double，結果為 2.5
```

### 2. 顯式型別轉換 (強制轉換)

使用 C++ 標準提供的 `static_cast<目標型別>(變數)` 進行安全轉換：

```cpp
int total = 10;
int count = 4;

// 傳統整數除法會截斷小數：10 / 4 結果為 2
// 透過 static_cast 強制轉為 double 後運算：
double average = static_cast<double>(total) / count; // 結果為 2.5
```

---

# 基本輸入與輸出 (I/O)

C++ 透過 `<iostream>` 標頭檔提供的串流物件進行資料互動。

## std::cout 標準輸出

用於在終端機輸出文字、變數與運算結果。

- `<<` 運算子可以連續串接多個輸出項目。
- `\n` 或 `std::endl` 用於換行（推薦平時使用 `\n`，效能較高）。

```cpp
#include <iostream>

int main() {
    int age = 18;
    std::string name = "Matthew";

    std::cout << "姓名: " << name << ", 年齡: " << age << "\n";
    return 0;
}
```

---

## std::cin 標準輸入

用於由鍵盤讀取使用者鍵入的資料，並存入指定變數中。

- `>>` 運算子以**空白字元（空格、Tab、換行）**作為資料分隔標記。

```cpp
#include <iostream>

int main() {
    int x = 0;
    int y = 0;

    std::cout << "請輸入兩個整數 (以空白分隔): ";
    std::cin >> x >> y; // 使用者輸入: 10 20

    std::cout << "兩數之和為: " << (x + y) << "\n";
    return 0;
}
```

---

## std::getline() 讀取整行字串 (含空格文字)

### 為什麼需要 std::getline()？(解決 cin >> 空格截斷問題)

- **傳統 `std::cin >> str;` 的限制**：
  
  `cin >>` 預設以**空白字元（空格、Tab、換行）**作為資料分隔標記。
  
  如果使用者在終端機輸入 `"Matthew Lin"`，`cin >>` 讀取到空格就會**中途停止截斷**，導致變數只拿到 `"Matthew"`，剩下的 `"Lin"` 則會殘留在緩衝區中造成後續程式錯亂！

- **`std::getline(std::cin, str);` 的解決方式**：
  
  `getline` 會**忽略中間的所有空格**，直到讀取到使用者按下 **Enter 鍵（換行符號 `\n`）** 為止，將整行文字完整存入字串中。

---

### 經典陷阱：為什麼 getline() 會被直接跳過？(緩衝區殘留問題)

這是所有 C++ 初學者最常遇到的程式 Bug：

```cpp
int age;
std::cin >> age; // 使用者輸入 20 並按下 Enter

std::string name;
std::getline(std::cin, name); // 程式當場跳過！根本沒停下來讓使用者輸入名字！
```

#### 底層記憶體水管（輸入緩衝區）解析：

```text
1. 使用者在鍵盤輸入「20」並按下「Enter」：
   輸入緩衝區內容：['2', '0', '\n']

2. 執行 std::cin >> age：
   cin 只拿走了整數「20」，但把換行符號「\n」留在了緩衝區中！
   輸入緩衝區殘留：['\n']

3. 接著執行 std::getline(std::cin, name)：
   getline 一看緩衝區開頭就是「\n」，以為使用者「輸入了空字串並按下了 Enter」，
   於是立刻結束讀取，導致 name 變成空字串 ""！
```

#### 正確解決解法：使用 std::cin.ignore() 清理換行符號

在 `cin >>` 之後、`std::getline()` 之前，呼叫 **`std::cin.ignore();`** 吞掉殘留在緩衝區內的 `\n`：

```cpp
#include <iostream>
#include <string>

int main() {
    int age = 0;
    std::string full_name;

    // 1. 讀取整數
    std::cout << "請輸入年齡: ";
    std::cin >> age;

    // 2. 關鍵步驟：清空留在緩衝區中的 Enter 換行符號
    std::cin.ignore();

    // 3. 正常讀取整行字串 (包含空格)
    std::cout << "請輸入完整姓名 (例如 Matthew Lin): ";
    std::getline(std::cin, full_name);

    // 4. 輸出驗證
    std::cout << "姓名: " << full_name << ", 年齡: " << age << "\n";
    return 0;
}
```

---

# 運算子與表達式

## 常用運算子分類總覽

### 1. 算術運算子

| 運算子 | 說明 | 範例 (`a = 10, b = 3`) | 結果 |
| :---: | :--- | :--- | :--- |
| `+` | 加法 | `a + b` | `13` |
| `-` | 減法 | `a - b` | `7` |
| `*` | 乘法 | `a * b` | `30` |
| `/` | 除法（整數相除會直接捨去小數） | `a / b` | `3` |
| `%` | 取餘數（模運算，只適用於整數） | `a % b` | `1` |

### 2. 遞增與遞減運算子

- `++a` (前置)：先將 `a` 加 1，再回傳加完後的值。
- `a++` (後置)：先回傳當前 `a` 的舊值，再將 `a` 加 1。

```cpp
int x = 5;
int y = ++x; // x 變為 6, y 拿到 6

int p = 5;
int q = p++; // q 拿到 5, p 變為 6
```

### 3. 關係與比較運算子

比較結果為 `bool` 型別（`true` 或 `false`）：

| 運算子 | 意義 | 範例 |
| :---: | :--- | :--- |
| `==` | 等於 | `a == b` |
| `!=` | 不等於 | `a != b` |
| `<` / `>` | 小於 / 大於 | `a < b` |
| `<=` / `>=` | 小於等於 / 大於等於 | `a <= b` |

### 4. 邏輯運算子

| 運算子 | 意義 | 說明 |
| :---: | :--- | :--- |
| `&&` | 邏輯且 (AND) | 兩邊條件都成立時才為 `true`（具備短路求值特性） |
| `\|\|` | 邏輯或 (OR) | 只要其中一邊條件成立即為 `true`（具備短路求值特性） |
| `!` | 邏輯非 (NOT) | 反轉真假值（`!true` 為 `false`） |

---

# 控制流程

## 條件判斷 (if / else)

根據條件表達式的真假，決定執行哪一段程式碼區塊。

```cpp
#include <iostream>

int main() {
    int score = 85;

    if (score >= 90) {
        std::cout << "等級: A\n";
    } else if (score >= 80) {
        std::cout << "等級: B\n";
    } else if (score >= 60) {
        std::cout << "等級: C\n";
    } else {
        std::cout << "等級: 不及格\n";
    }
    return 0;
}
```

---

## switch-case 多分支選擇

- **使用時機**：針對單一整數或字元變數進行多個固定數值的等值比對。
- **重要規則**：每個 `case` 結尾通常需加上 `break;` 跳出分支，否則會繼續向下執行後續的 case（稱為 Fall-through 現象）。

```cpp
#include <iostream>

int main() {
    char op = '+';
    int a = 10, b = 5;

    switch (op) {
        case '+':
            std::cout << "結果: " << (a + b) << "\n";
            break;
        case '-':
            std::cout << "結果: " << (a - b) << "\n";
            break;
        case '*':
            std::cout << "結果: " << (a * b) << "\n";
            break;
        default:
            std::cout << "未知運算子\n";
            break;
    }
    return 0;
}
```

---

## 迴圈結構

### 1. while 迴圈

當條件成立時重複執行，適合**執行次數未預先確定**的情境。

```cpp
int count = 1;
while (count <= 5) {
    std::cout << count << " ";
    count++;
}
// 輸出: 1 2 3 4 5
```

### 2. for 迴圈

由「初始值設定」、「迴圈終止條件」與「每次迭代更新」組成，適合**已知執行次數**的情境。

```cpp
for (int i = 0; i < 5; i++) {
    std::cout << "第 " << i << " 次迭代\n";
}
```

### 3. break 與 continue

- `break`：立即強制終止並跳出整個迴圈。
- `continue`：立即跳過本次迴圈剩餘程式碼，直接進入下一次迭代。

```cpp
for (int i = 1; i <= 5; i++) {
    if (i == 3) continue; // 跳過 3
    if (i == 5) break;    // 遇到 5 終止迴圈
    std::cout << i << " ";
}
// 輸出: 1 2 4
```

---

# 函式 (Functions)

函式是將特定功能的程式碼封裝成可重複呼叫的獨立區塊。

## 函式定義基本結構

```cpp
回傳型別 函式名稱(參數型別 參數名稱1, 參數型別 參數名稱2) {
    // 執行的程式碼
    return 回傳值;
}
```

```cpp
#include <iostream>

// 定義一個計算兩數相加的函式
int add(int a, int b) {
    return a + b;
}

int main() {
    int result = add(3, 5);
    std::cout << "相加結果: " << result << "\n"; // 輸出: 8
    return 0;
}
```

---

## 參數傳遞機制：傳值 vs 傳參考

C++ 支援多種參數傳遞方式，對效能與資料修改有決定性影響：

### 1. 傳值 (Pass by Value)

- **核心概念**：
  
  當你呼叫函式並把變數傳進去時，電腦會在記憶體中**「影印一份全新的副本」**交給函式。
  
  函式內部的參數 `x` 拿到的只是這張**影印紙**，在函式內對 `x` 做任何修改，**完全不會影響到外部原變數（正本）的值**！

#### 白話影印本比喻與記憶體圖解：

```text
1. main() 建立變數：
   [正本 num] (記憶體位址 A) = 10

2. 呼叫 modify_value(num)：
   電腦把 10 影印一份，存入新變數 [副本 x] (記憶體位址 B) = 10

3. 函式內部執行 x = 100：
   [正本 num] (位址 A) = 10   <--- 完全沒被碰到！
   [副本 x]   (位址 B) = 100  <--- 改的只是這張影印紙！

4. 函式執行完畢結束：
   [副本 x] 被丟進垃圾桶銷毀，回到 main() 時 [正本 num] 依然是 10。
```

```cpp
#include <iostream>

void modify_value(int x) {
    x = 100; // 只修改了副本 x (位址 B)，正本 num (位址 A) 毫髮無傷
}

int main() {
    int num = 10;
    modify_value(num); // 傳入 num 的數值副本
    std::cout << num << "\n"; // 輸出仍為 10！
    return 0;
}
```

### 2. 傳參考 (Pass by Reference `&`)

- **核心概念**：
  
  函式參數**不會開闢新的記憶體空間**，而是**直接共用外部原變數的記憶體位址**（`&x` 完全等於 `&num`）。
  
  此時的 `x` 就只是 `num` 的另一個**「綽號 / 別名 (Alias)」**，在函式內改動 `x`，就是直接改動正本！

#### 記憶體位址共用圖解：

```text
1. main() 建立變數：
   [正本 num] 住在記憶體位址 0x7ffd01 (值為 10)

2. 呼叫 modify_reference(num)：
   函式的參數 x 直接綁定到位址 0x7ffd01（完全沒有影印！）

3. 函式內部執行 x = 100：
   直接把位址 0x7ffd01 裡面的數值改成 100！

4. 函式結束回到 main()：
   位址 0x7ffd01 的 num 已經真正變成了 100。
```

```cpp
#include <iostream>

void modify_reference(int& x) {
    std::cout << "函式內 x 的記憶體位址:   " << &x << "\n";
    x = 100; // 直接改動這塊記憶體的值
}

int main() {
    int num = 10;
    std::cout << "main 內 num 的記憶體位址: " << &num << "\n";

    modify_reference(num); // 傳入引用 (共用位址)

    std::cout << "呼叫後 num 的值變為: " << num << "\n"; // 輸出變為 100！
    return 0;
}
```

### 3. 傳常數參考 (Pass by Const Reference `const &`)

**現代 C++ 傳參最佳實踐**：既享有「不拷貝資料」的高效能，又加上 `const` 保證函式內部「絕不能意外修改外部資料」。

```cpp
void print_vector(const std::vector<int>& vec) {
    for (int n : vec) {
        std::cout << n << " ";
    }
    // vec.push_back(10); // 編譯錯誤：不可修改 const 參考
}
```

---

## 函式多載 (Function Overloading)

在同一個作用域中，可以定義**名稱完全相同，但參數個數或參數型別不同**的多個函式：

```cpp
#include <iostream>

int multiply(int a, int b) {
    return a * b;
}

double multiply(double a, double b) {
    return a * b;
}

int main() {
    std::cout << multiply(2, 3) << "\n";       // 自動呼叫 int 版本，輸出 6
    std::cout << multiply(2.5, 4.0) << "\n";   // 自動呼叫 double 版本，輸出 10.0
    return 0;
}
```

---

# 指標與記憶體管理入門

指標是 C++ 最強大也最需要謹慎對待的核心機制。

## 什麼是記憶體位址？

程式中宣告的每一個變數，在記憶體中都有一個專屬的門牌號碼，稱為**記憶體位址**。

使用**取位址運算子 `&`** 可以取得變數的記憶體位址：

```cpp
#include <iostream>

int main() {
    int score = 100;
    std::cout << "變數數值: " << score << "\n";
    std::cout << "記憶體位址: " << &score << "\n"; // 輸出十六進位位址，如 0x7ffee4b288bc
    return 0;
}
```

---

## 指標變數與提領運算子

- **指標變數**：專門用來存放「另一個變數之記憶體位址」的變數。型別宣告為 `型別*`。
- **提領運算子 `*`**：透過指標變數內部存取的位址，直接存取或修改該位址上的真實資料。

```cpp
#include <iostream>

int main() {
    int val = 42;
    int* ptr = &val; // ptr 指向 val 的記憶體位址

    std::cout << "ptr 儲存的位址: " << ptr << "\n";
    std::cout << "透過 *ptr 讀取數值: " << *ptr << "\n"; // 輸出: 42

    // 透過指標修改原變數
    *ptr = 99;
    std::cout << "修改後 val 的數值: " << val << "\n";   // 輸出: 99
    return 0;
}
```

---

## 空指標 nullptr

當一個指標尚未指向任何有效記憶體位址時，**務必初始化為 `nullptr`**。

在提領指標前，應養成進行非空檢查的好習慣：

```cpp
int* p = nullptr; // 安全初始化為空指標

if (p != nullptr) {
    std::cout << *p << "\n";
} else {
    std::cout << "指標目前為空，不可提領讀取！\n";
}
```

---

## 指標 (Pointer) 與 參考 (Reference) 差異對照表

| 比較維度 | 指標 (Pointer `*`) | 參考 (Reference `&`) |
| :--- | :--- | :--- |
| **本質** | 獨立變數，內部儲存目標記憶體位址 | 現存變數的永久別名 |
| **是否可為空** | **可以** (`nullptr`) | **絕對不行**（宣告時必須立刻綁定實體變數） |
| **是否可重新指向** | **可以**（可隨時改變儲存的位址） | **不行**（一生只能綁定最初的變數） |
| **使用語法** | 需透過 `*p` 提領存取 | 如原生變數般直接使用 `ref` |

---

# 物件導向程式設計基礎 (OOP)

## 結構體 struct 與 類別 class

用來將關聯的資料屬性與函式方法封裝成自訂的複合型別。

- `struct`：內部成員預設權限為 **`public`（公開）**。
- `class`：內部成員預設權限為 **`private`（私有，外部無法直接讀寫）**。

### 三大存取權限修飾詞

1. **`public`**：任何外部程式碼皆可自由讀寫。
2. **`private`**：只有類別內部的成員函式可以存取。
3. **`protected`**：僅自己與繼承的子類別可以存取。

---

## 類別封裝與建構子範例

```cpp
#include <iostream>
#include <string>

class Student {
private:
    std::string name_; // 私有屬性，防止外部惡意篡改
    int score_ = 0;

public:
    // 建構子 (Constructor)：在建立物件時自動執行初始化
    Student(std::string name, int score) : name_(name), score_(score) {
        std::cout << "建立學生物件: " << name_ << "\n";
    }

    // 解構子 (Destructor)：物件生命週期結束銷毀時自動執行
    ~Student() {
        std::cout << "銷毀學生物件: " << name_ << "\n";
    }

    // 公有方法 (Getter & Setter)
    void print_info() const {
        std::cout << "學生: " << name_ << ", 成績: " << score_ << "\n";
    }

    void set_score(int new_score) {
        if (new_score >= 0 && new_score <= 100) {
            score_ = new_score;
        }
    }
};

int main() {
    // 建立 Student 物件實例
    Student s1("Alice", 95);
    s1.print_info();

    s1.set_score(98);
    s1.print_info();

    return 0; // s1 在 main 結束時自動觸發解構子銷毀
}
```

---

# 核心功能與常用標準函式大解密

##### main() 程式入口主函數

- **使用時機**：每一個獨立執行的 C++ 程式都必須且只能有一個 `main()` 函式。
- **語法**：
  ```cpp
  int main() {
      // 程式邏輯
      return 0;
  }
  ```
- **回傳值**：
  - `int`：返回給作業系統的狀態碼（`0` 代表執行成功，非 0 代表異常錯誤碼）。

```cpp
#include <iostream>

int main() {
    std::cout << "程式順利啟動並執行完成。\n";
    return 0;
}
```

---

## 一、 數學計算常用函式 (<cmath> 與 <algorithm>)

##### std::sqrt() 計算平方根

- **使用時機**：計算某個數值的平方根（開根號）。
- **所屬標頭檔**：`#include <cmath>`
- **語法**：`double std::sqrt(double x);`
- **參數說明**：
  - `x`：要開平方根的數值（不能為負數）。
- **回傳值**：
  - `double`：計算後的平方根結果。

```cpp
#include <iostream>
#include <cmath>

int main() {
    double result = std::sqrt(25.0);
    std::cout << "25 的平方根: " << result << "\n"; // 輸出: 5
    return 0;
}
```

---

##### std::pow() 計算次方次冪

- **使用時機**：計算底數的指數次方（例如 $x^y$）。
- **所屬標頭檔**：`#include <cmath>`
- **語法**：`double std::pow(double base, double exp);`
- **參數說明**：
  - `base`：底數。
  - `exp`：指數（次方數）。
- **回傳值**：
  - `double`：$base^{exp}$ 的計算結果。

```cpp
#include <iostream>
#include <cmath>

int main() {
    double result = std::pow(2.0, 3.0); // 計算 2 的 3 次方
    std::cout << "2 的 3 次方: " << result << "\n"; // 輸出: 8
    return 0;
}
```

---

##### std::abs() 計算絕對值

- **使用時機**：取得數值的非負絕對值。
- **所屬標頭檔**：`#include <cmath>` (浮點數) 或 `#include <cstdlib>` (整數)
- **語法**：`int std::abs(int x);` 或 `double std::abs(double x);`
- **參數說明**：
  - `x`：輸入的整數或浮點數。
- **回傳值**：
  - 輸入數值的絕對值。

```cpp
#include <iostream>
#include <cmath>

int main() {
    int val = -42;
    std::cout << "絕對值: " << std::abs(val) << "\n"; // 輸出: 42
    return 0;
}
```

---

##### std::max() 與 std::min() 取最大值與最小值

- **使用時機**：快速比較並取得兩數（或多個數值清單）中較大或較小者。
- **所屬標頭檔**：`#include <algorithm>`
- **語法**：
  - `std::max(a, b);` 或 `std::max({a, b, c, d});`
  - `std::min(a, b);` 或 `std::min({a, b, c, d});`
- **回傳值**：
  - 比較後的極大值或極小值。

```cpp
#include <iostream>
#include <algorithm>

int main() {
    int a = 15, b = 27;
    std::cout << "較大值: " << std::max(a, b) << "\n"; // 輸出: 27
    std::cout << "較小值: " << std::min(a, b) << "\n"; // 輸出: 15

    // 支援以大括號清單比較多個數值：
    int largest = std::max({3, 9, 24, 12, 8});
    std::cout << "清單最大值: " << largest << "\n"; // 輸出: 24
    return 0;
}
```

---

##### std::round() / std::floor() / std::ceil() 數值取整

- **使用時機**：浮點數取整處理。
  - `std::round()`：**四捨五入**到最接近的整數。
  - `std::floor()`：**無條件捨去**（向下取整）。
  - `std::ceil()`：**無條件進位**（向上取整）。
- **所屬標頭檔**：`#include <cmath>`

```cpp
#include <iostream>
#include <cmath>

int main() {
    double num = 3.6;

    std::cout << "四捨五入 round(3.6):   " << std::round(num) << "\n"; // 輸出: 4
    std::cout << "無條件捨去 floor(3.6):   " << std::floor(num) << "\n"; // 輸出: 3
    std::cout << "無條件進位 ceil(3.6):    " << std::ceil(num) << "\n";  // 輸出: 4
    return 0;
}
```

---

## 二、 字串與數值互轉函式 (<string>)

##### std::to_string() 數值轉為字串

- **使用時機**：將整數或浮點數（如 `int`, `double`）轉換為標準文字字串 `std::string`，以便進行字串串接。
- **所屬標頭檔**：`#include <string>`
- **語法**：`std::string std::to_string(數值);`
- **回傳值**：
  - `std::string`：轉換後的字串。

```cpp
#include <iostream>
#include <string>

int main() {
    int score = 95;
    std::string msg = "你的成績是: " + std::to_string(score) + " 分";
    std::cout << msg << "\n"; // 輸出: 你的成績是: 95 分
    return 0;
}
```

---

##### std::stoi() 與 std::stod() 字串轉整數與浮點數

- **使用時機**：將字串解析並轉換為數值型別。
  - `std::stoi()`：String to Integer（字串轉 `int`）。
  - `std::stod()`：String to Double（字串轉 `double`）。
  - `std::stoll()`：String to Long Long（字串轉 `long long`）。
- **所屬標頭檔**：`#include <string>`

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str_int = "123";
    std::string str_double = "3.1415";

    int num = std::stoi(str_int);
    double pi = std::stod(str_double);

    std::cout << "整數加 1: " << (num + 1) << "\n"; // 輸出: 124
    std::cout << "浮點乘 2: " << (pi * 2) << "\n";  // 輸出: 6.283
    return 0;
}
```

---

## 三、 演算法與容器操作函式 (<algorithm>)

##### std::sort() 快速排序演算法

- **使用時機**：對 `std::vector` 或原生陣列進行由小到大（升序）或自訂規則的快速排序（平均時間複雜度為 $O(N \log N)$）。
- **所屬標頭檔**：`#include <algorithm>`
- **語法**：
  - `std::sort(vec.begin(), vec.end());`（預設升序）
  - `std::sort(vec.rbegin(), vec.rend());`（降序，由大到小）

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {5, 2, 8, 1, 9};

    // 升序排序 (由小到大)
    std::sort(nums.begin(), nums.end());

    std::cout << "排序後: ";
    for (int n : nums) std::cout << n << " "; // 輸出: 1 2 5 8 9
    std::cout << "\n";
    return 0;
}
```

---

##### std::reverse() 元素順序反轉

- **使用時機**：原地將容器或字串的元素順序前後顛倒反轉。
- **所屬標頭檔**：`#include <algorithm>`
- **語法**：`std::reverse(起始迭代器, 結束迭代器);`

```cpp
#include <iostream>
#include <string>
#include <algorithm>

int main() {
    std::string str = "ABCDE";
    std::reverse(str.begin(), str.end());
    std::cout << "反轉後字串: " << str << "\n"; // 輸出: EDCBA
    return 0;
}
```

---

##### std::find() 線性搜尋元素

- **使用時機**：在容器中尋找特定數值第一次出現的位置。
- **所屬標頭檔**：`#include <algorithm>`
- **語法**：`auto it = std::find(vec.begin(), vec.end(), 目標值);`
- **回傳值**：
  - 若找到：回傳指向該元素的迭代器（Iterator）。
  - 若未找到：回傳容器結尾迭代器 `vec.end()`。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> nums = {10, 20, 30, 40};

    auto it = std::find(nums.begin(), nums.end(), 30);

    if (it != nums.end()) {
        std::cout << "成功找到數值 30！\n";
    } else {
        std::cout << "找不到該數值。\n";
    }
    return 0;
}
```

---

##### std::count() 統計元素出現次數

- **使用時機**：計算某個特定數值在容器中出現的總次數。
- **所屬標頭檔**：`#include <algorithm>`
- **語法**：`int cnt = std::count(vec.begin(), vec.end(), 目標值);`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> scores = {90, 80, 90, 70, 90};

    int count_90 = std::count(scores.begin(), scores.end(), 90);
    std::cout << "獲得 90 分的人數: " << count_90 << "\n"; // 輸出: 3
    return 0;
}
```

---

## 四、 字元檢查與大小寫轉換 (<cctype>)

##### std::isdigit() 與 std::isalpha() 字元檢查

- **使用時機**：判斷單一字元 `char` 的屬性。
  - `std::isdigit(c)`：檢查字元是否為**阿拉伯數字 (`'0'`~`'9'`)**。
  - `std::isalpha(c)`：檢查字元是否為**英文字母 (`'a'`~`'z'`, `'A'`~`'Z'`)**。
- **所屬標頭檔**：`#include <cctype>`

```cpp
#include <iostream>
#include <cctype>

int main() {
    char c1 = '7';
    char c2 = 'A';

    if (std::isdigit(c1)) std::cout << c1 << " 是數字\n"; // 輸出: 7 是數字
    if (std::isalpha(c2)) std::cout << c2 << " 是字母\n"; // 輸出: A 是字母
    return 0;
}
```

---

##### std::tolower() 與 std::toupper() 英文字母大小寫轉換

- **使用時機**：將單一英文字元轉換為小寫或大寫。
- **所屬標頭檔**：`#include <cctype>`

```cpp
#include <iostream>
#include <cctype>

int main() {
    char lower = std::tolower('G'); // 轉換為小寫 'g'
    char upper = std::toupper('b'); // 轉換為大寫 'B'

    std::cout << "小寫: " << lower << ", 大寫: " << upper << "\n"; // 輸出: 小寫: g, 大寫: B
    return 0;
}
```

---

## 五、 隨機數生成 (<cstdlib> 與 <ctime>)

##### rand() 與 srand() 產生隨機數

- **使用時機**：在遊戲、模擬或抽籤中生成隨機整數。
- **所屬標頭檔**：`#include <cstdlib>` 與 `#include <ctime>`
- **運作核心**：
  - `srand(time(nullptr));`：以當前時間作為**隨機數種子 (Seed)** 進行初始化（只需在程式開頭執行一次）。
  - `rand() % N;`：產生範圍在 `0 ~ N-1` 之間的隨機整數。

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

int main() {
    // 1. 初始化隨機數種子 (確保每次執行結果不同)
    std::srand(std::time(nullptr));

    // 2. 產生 1 到 6 的隨機骰子點數 (rand() % 6 產生 0~5，加 1 變 1~6)
    int dice = (std::rand() % 6) + 1;
    std::cout << "擲出骰子點數: " << dice << "\n";
    return 0;
}
```

---

# 實戰除錯與初學者核心天條

## 1. 變數未初始化天條

> **未初始化變數天條**：  
> 在 C++ 中，區域變數宣告時若未給定初始值（例如 `int count;`），它的初始內容將是**記憶體殘留的隨機垃圾值**！  
> **鐵律**：宣告變數時務必立即給予初始值，例如 `int count = 0;` 或 `double price = 0.0;`。

---

## 2. 陣列越界存取天條

> **陣列越界天條**：  
> C++ 原生陣列不會自動檢查索引範圍。若陣列長度為 5（合法索引 0~4），存取 `arr[5]` 或 `arr[10]` 將踩到其他變數的記憶體，引發難以追蹤的未定義行為。  
> **鐵律**：平時一律優先使用 `std::vector`，需要確保安全時可使用 `vec.at(index)` 進行自動邊界檢查。

---

## 3. 野指標與空指標提領天條

> **空指標提領天條**：  
> 提領一個未初始化或值為 `nullptr` 的指標（例如 `int* p = nullptr; *p = 10;`）會導致作業系統直接強制終止程式（Segmentation Fault 崩潰）。  
> **鐵律**：  
> 1. 指標宣告時一律初始化為 `nullptr`。  
> 2. 提領前必須先判斷 `if (p != nullptr)`。

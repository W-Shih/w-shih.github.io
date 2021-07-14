# Open–closed principle - 二元計算器
[toc]

## 什麼是 Open–closed principle?
- Open–closed principle:
    > 對擴展開放, 對修改封閉
- 增加功能時, 應盡量
  - 採用代碼擴展: 以增加新的類模塊的方式進行擴展。
  - 避免對原有的代碼進行修改: 避免直接修改已存在的模塊, 而產生重新編譯, 重新佈署, 重新測試等問題。

## 範例 - 二元計算器
### 設想場景
-  在設計時, 要先預想未來如果需要新增一個 operator 時, 代碼的變動方式。
-  在學習設計模式時, 要預先考慮未來的變化, 才更能體會設計模式的精隨。
  
### 不符合 Open–closed principle 的設計方式
- 先看不符合 Open–closed principle 的設計方式:
  - 如果需要新增一個 mod operator 時, 代碼的變動方式為更改 `Calculator::getResult()`
    ```cpp
    #include <iostream>
    #include <string>
    using namespace std;

    // ---------------------------------------------------------------------
    class Calculator {
    public:
        Calculator(int a, int b, string op) : m_a(a), m_b(b), m_op(op) {}
        
        virtual ~Calculator() {
            cout << "Calculator: destructor..." << endl;
        }
        
        pair<pair<int, int>, string> getter() const {
            return {{m_a, m_b}, m_op};
        };
        
        int getResult() const {
            if (m_op == "+") {
                return m_a + m_b;
            }
            else {
                return INT32_MAX;
            }
        }
    private:
        int m_a;
        int m_b;
        string m_op;
    };

    // ---------------------------------------------------------------------
    int main() // Client
    {
        Calculator* calculator = new Calculator(3, 5, "+");
        cout << calculator->getter().first.first << ", " 
             << calculator->getter().first.second << ", " 
             << calculator->getter().second << endl;
        cout << calculator->getResult() << endl;
        delete calculator;
        calculator = nullptr;
        cout << "------------------------------" << endl;
        
        calculator = new Calculator(3, 5, "%");
        cout << calculator->getter().first.first << ", " 
             << calculator->getter().first.second << ", " 
             << calculator->getter().second << endl;
        cout << calculator->getResult() << endl;
        delete calculator;
        calculator = nullptr;
        
        return 0;
    }

    ```

  - 執行結果:
    ```text
    3, 5, +
    8
    Calculator: destructor...
    ------------------------------
    3, 5, %
    2147483647
    Calculator: destructor...
    ```
- 備註: 同時, `class Calculator` 的責任過重, `Calculator::getResult()` 負責各種二元計算, 也違反了 *Single Responsibility Priciple*。。

### 符合 Open–closed principle 的設計方式
- 再看符合 Open–closed principle 的設計方式:
  - 此時如果需要新增一個 mod operator 時, 代碼的變動方式為新增 `class ModCalculator`
    ```cpp
    #include <iostream>
    using namespace std;

    // ---------------------------------------------------------------------
    class ICalculator {
    public:
        virtual int getResult() = 0;
        virtual void setAB(int a, int b) = 0;
        virtual pair<int, int> getAB() const = 0;
        // destructor 一定要為 virtual
        // OW, 子類的 destructor 不會被執行
        virtual ~ICalculator() {
            cout << "ICalculator: destructor..." << endl;
        };
    protected:
        int m_a;
        int m_b;
    };

    // ---------------------------------------------------------------------
    class AdditionCalculator : public ICalculator {
        void setAB(int a, int b) final {
            m_a = a;
            m_b = b;
        }
        
        pair<int, int> getAB() const final {
            return {m_a, m_b};
        }
        
        int getResult() final {
            return m_a + m_b;
        }
        
        ~AdditionCalculator() {
            cout << "AdditionCalculator: destructor..." << endl;
        }
    };

    // ---------------------------------------------------------------------
    class ModCalculator : public ICalculator {
        void setAB(int a, int b) final {
            m_a = a;
            m_b = b;
        }
        
        pair<int, int> getAB() const final {
            return {m_a, m_b};
        }
        
        int getResult() final {
            return m_a % m_b;
        }
        
        ~ModCalculator() {
            cout << "ModCalculator: destructor..." << endl;
        }
    };

    // ---------------------------------------------------------------------
    int main() // Client
    {
        ICalculator* calculator = new AdditionCalculator();
        calculator->setAB(3, 5);
        cout << calculator->getAB().first << ", " 
             << calculator->getAB().second << endl;
        cout << calculator->getResult() << endl;
        delete calculator;
        calculator = nullptr;
        cout << "------------------------------" << endl;
        
        calculator = new ModCalculator();
        calculator->setAB(13, 3);
        cout << calculator->getAB().first << ", " 
             << calculator->getAB().second << endl;
        cout << calculator->getResult() << endl;;
        delete calculator;
        calculator = nullptr;
        
        return 0;
    }

    ```
  - 執行結果:
    ```text
    3, 5
    8
    AdditionCalculator: destructor...
    ICalculator: destructor...
    ------------------------------
    13, 3
    1
    ModCalculator: destructor...
    ICalculator: destructor...
    ```

## C++ 小學堂
- [C++11 中 final 和 override 的用法](https://blog.csdn.net/jirryzhang/article/details/82961654)
- [比較安全的 C++ 虛擬函式寫法：C++11 override 與 final](https://kheresy.wordpress.com/2014/10/03/override-and-final-in-cpp-11/)
- [C++ final 關鍵字](https://blog.csdn.net/mayue_web/article/details/88406527)

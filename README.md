#include <iostream>
#include <cstring>
#include <cctype>
using namespace std;

class LString {
    struct Buffer {
        char* data;
        int refCount;
        Buffer(const char* str) {
            data = new char[strlen(str) + 1];
            strcpy(data, str);
            refCount = 1;
        }
        ~Buffer() { delete[] data; }
    };

    Buffer* m_pBuff;

    // Copy-on-write helper
    void detach() {
        if (m_pBuff && m_pBuff->refCount > 1) {
            m_pBuff->refCount--;
            m_pBuff = new Buffer(m_pBuff->data);
        }
    }

public:
    // Constructors
    LString(const char* str = "") {
        m_pBuff = new Buffer(str);
    }

    LString(const LString& other) {
        m_pBuff = other.m_pBuff;
        m_pBuff->refCount++;
    }

    // Destructor
    ~LString() {
        if (--m_pBuff->refCount == 0) {
            delete m_pBuff;
        }
    }

    // Assignment
    LString& operator=(const LString& other) {
        if (this != &other) {
            if (--m_pBuff->refCount == 0)
                delete m_pBuff;
            m_pBuff = other.m_pBuff;
            m_pBuff->refCount++;
        }
        return *this;
    }

    LString& operator=(const char* str) {
        if (--m_pBuff->refCount == 0)
            delete m_pBuff;
        m_pBuff = new Buffer(str);
        return *this;
    }

    // Getters
    const char* GetString() const { return m_pBuff->data; }
    int GetLength() const { return strlen(m_pBuff->data); }

    // [] operator (read-only)
    char operator[](int index) const {
        if (index >= 0 && index < GetLength())
            return m_pBuff->data[index];
        throw out_of_range("Index out of range");
    }

    // SetString
    void SetString(const char* str) {
        detach();
        delete[] m_pBuff->data;
        m_pBuff->data = new char[strlen(str) + 1];
        strcpy(m_pBuff->data, str);
    }

    // Insert a string at index
    void Insert(int index, const char* str) {
        if (index < 0 || index > GetLength()) return;
        detach();
        int newLen = GetLength() + strlen(str);
        char* newData = new char[newLen + 1];
        strncpy(newData, m_pBuff->data, index);
        newData[index] = '\0';
        strcat(newData, str);
        strcat(newData, m_pBuff->data + index);
        delete[] m_pBuff->data;
        m_pBuff->data = newData;
    }

    // Insert a character multiple times
    void Insert(int index, char ch, int count) {
        if (index < 0 || index > GetLength()) return;
        detach();
        int newLen = GetLength() + count;
        char* newData = new char[newLen + 1];
        strncpy(newData, m_pBuff->data, index);
        for (int i = 0; i < count; i++)
            newData[index + i] = ch;
        strcpy(newData + index + count, m_pBuff->data + index);
        delete[] m_pBuff->data;
        m_pBuff->data = newData;
    }

    // Remove a character at index
    void Remove(int index) {
        if (index < 0 || index >= GetLength()) return;
        detach();
        int len = GetLength();
        char* newData = new char[len];
        strncpy(newData, m_pBuff->data, index);
        strcpy(newData + index, m_pBuff->data + index + 1);
        delete[] m_pBuff->data;
        m_pBuff->data = newData;
    }

    // Remove all occurrences of a char
    void RemoveAll(char ch) {
        detach();
        int len = GetLength();
        char* newData = new char[len + 1];
        int j = 0;
        for (int i = 0; i < len; i++) {
            if (m_pBuff->data[i] != ch)
                newData[j++] = m_pBuff->data[i];
        }
        newData[j] = '\0';
        delete[] m_pBuff->data;
        m_pBuff->data = newData;
    }

    // Trim left n characters
    void TrimLeft(int n) {
        if (n <= 0 || n > GetLength()) return;
        detach();
        char* newData = new char[GetLength() - n + 1];
        strcpy(newData, m_pBuff->data + n);
        delete[] m_pBuff->data;
        m_pBuff->data = newData;
    }

    // Trim right n characters
    void TrimRight(int n) {
        if (n <= 0 || n > GetLength()) return;
        detach();
        m_pBuff->data[GetLength() - n] = '\0';
    }

    // Find substring
    int Find(const char* str) const {
        char* pos = strstr(m_pBuff->data, str);
        return pos ? (pos - m_pBuff->data) : -1;
    }

    // Find whole word
    int FindWholeWord(const char* word) const {
        const char* text = m_pBuff->data;
        int wordLen = strlen(word);
        for (int i = 0; text[i]; i++) {
            if (strncmp(&text[i], word, wordLen) == 0) {
                char before = (i == 0) ? ' ' : text[i - 1];
                char after = text[i + wordLen];
                if (isspace(before) || ispunct(before)) {
                    if (after == '\0' || isspace(after) || ispunct(after))
                        return i;
                }
            }
        }
        return -1;
    }

    // Concatenation
    LString operator+(const LString& other) const {
        int newLen = GetLength() + other.GetLength();
        char* newData = new char[newLen + 1];
        strcpy(newData, m_pBuff->data);
        strcat(newData, other.m_pBuff->data);
        LString result(newData);
        delete[] newData;
        return result;
    }

    LString operator+(const char* str) const {
        int newLen = GetLength() + strlen(str);
        char* newData = new char[newLen + 1];
        strcpy(newData, m_pBuff->data);
        strcat(newData, str);
        LString result(newData);
        delete[] newData;
        return result;
    }

    // += operators
    LString& operator+=(const LString& other) {
        *this = *this + other;
        return *this;
    }

    LString& operator+=(const char* str) {
        *this = *this + str;
        return *this;
    }

    LString& operator+=(char ch) {
        char temp[2] = { ch, '\0' };
        *this = *this + temp;
        return *this;
    }

    // Comparison
    bool operator==(const LString& other) const {
        return strcmp(m_pBuff->data, other.m_pBuff->data) == 0;
    }

    bool operator==(const char* str) const {
        return strcmp(m_pBuff->data, str) == 0;
    }

    bool operator<(const LString& other) const {
        return strcmp(m_pBuff->data, other.m_pBuff->data) < 0;
    }

    bool operator>(const LString& other) const {
        return strcmp(m_pBuff->data, other.m_pBuff->data) > 0;
    }
};

// Print
ostream& operator<<(ostream& os, const LString& s) {
    os << s.GetString();
    return os;
}

// ---------------- Main with Examples ----------------
int main() {
    LString s1 = "C++";
    LString s2(s1);   // shares buffer
    cout << "s1: " << s1 << ", s2: " << s2 << endl;

    s1 = "Language";  // s1 gets new buffer, s2 still = "C++"
    cout << "After s1 change -> s1: " << s1 << ", s2: " << s2 << endl;

    LString s3(s2), s4(s2); // s3, s4 share with s2
    cout << "s2, s3, s4: " << s2 << ", " << s3 << ", " << s4 << endl;

    s3 = s1; // now s3 points to "Language"
    cout << "s3 = s1 -> s3: " << s3 << ", s1: " << s1 << endl;

    LString s5 = "C++"; // shares with s2
    cout << "s5: " << s5 << " (shares with s2)" << endl;

    s3.SetString("C++"); // copy-on-write
    cout << "After SetString, s3: " << s3 << ", s2: " << s2 << endl;

    // Insert
    s3.Insert(3, " is fun");
    cout << "After Insert: " << s3 << endl;

    // Insert char
    s3.Insert(0, '*', 3);
    cout << "Insert *** at start: " << s3 << endl;

    // Remove
    s3.Remove(0);
    cout << "After Remove index 0: " << s3 << endl;

    // RemoveAll
    s3.RemoveAll('*');
    cout << "After RemoveAll '*': " << s3 << endl;

    // TrimLeft and TrimRight
    LString s6 = "   Hello World!!!   ";
    cout << "s6 before Trim: [" << s6 << "]" << endl;
    s6.TrimLeft(3);
    s6.TrimRight(3);
    cout << "s6 after Trim: [" << s6 << "]" << endl;

    // Find
    LString s7 = "C++ Language is powerful";
    cout << "Find 'Language': " << s7.Find("Language") << endl;
    cout << "Find 'Java': " << s7.Find("Java") << endl;

    // FindWholeWord
    cout << "FindWholeWord 'is': " << s7.FindWholeWord("is") << endl;

    // Concatenation
    LString s8 = s7 + " indeed!";
    cout << "Concatenated: " << s8 << endl;

    // += operators
    s8 += " :)";
    s8 += '!';
    cout << "After += : " << s8 << endl;

    // Comparison
    cout << (s1 == "Language" ? "s1 == Language" : "Not equal") << endl;
    cout << (s2 < s1 ? "s2 < s1" : "s2 >= s1") << endl;

    return 0;
}

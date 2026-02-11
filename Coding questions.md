Below are **concise, optimal C++ solutions** for each problem you listed.
(All use standard headers and optimal time complexity.)

---

# 🔹 Arrays & Strings

## 1️⃣ Two Sum (O(n))

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> mp;
    for(int i = 0; i < nums.size(); i++) {
        int complement = target - nums[i];
        if(mp.count(complement))
            return {mp[complement], i};
        mp[nums[i]] = i;
    }
    return {};
}
```

---

## 2️⃣ Longest Substring Without Repeating Characters (O(n))

```cpp
int lengthOfLongestSubstring(string s) {
    vector<int> index(256, -1);
    int left = 0, maxLen = 0;
    for(int right = 0; right < s.size(); right++) {
        if(index[s[right]] >= left)
            left = index[s[right]] + 1;
        index[s[right]] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## 3️⃣ Merge Intervals (O(n log n))

```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    vector<vector<int>> result;
    for(auto &interval : intervals) {
        if(result.empty() || result.back()[1] < interval[0])
            result.push_back(interval);
        else
            result.back()[1] = max(result.back()[1], interval[1]);
    }
    return result;
}
```

---

## 4️⃣ Product of Array Except Self (O(n))

```cpp
vector<int> productExceptSelf(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n,1);
    int prefix = 1;
    for(int i=0;i<n;i++){
        res[i] = prefix;
        prefix *= nums[i];
    }
    int suffix = 1;
    for(int i=n-1;i>=0;i--){
        res[i] *= suffix;
        suffix *= nums[i];
    }
    return res;
}
```

---

# 🔹 Hashing

## 5️⃣ Group Anagrams

```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> mp;
    for(string s : strs){
        string key = s;
        sort(key.begin(), key.end());
        mp[key].push_back(s);
    }
    vector<vector<string>> result;
    for(auto &p : mp)
        result.push_back(p.second);
    return result;
}
```

---

## 6️⃣ First Non-Repeating Character

```cpp
int firstUniqChar(string s) {
    vector<int> freq(26,0);
    for(char c : s) freq[c-'a']++;
    for(int i=0;i<s.size();i++)
        if(freq[s[i]-'a']==1) return i;
    return -1;
}
```

---

# 🔹 Linked List

(Assume standard ListNode definition)

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(NULL) {}
};
```

## 7️⃣ Reverse Linked List

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode *prev = NULL, *curr = head;
    while(curr){
        ListNode* next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

---

## 8️⃣ Detect Cycle (Floyd’s)

```cpp
bool hasCycle(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while(fast && fast->next){
        slow = slow->next;
        fast = fast->next->next;
        if(slow == fast) return true;
    }
    return false;
}
```

---

## 9️⃣ Merge Two Sorted Lists

```cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while(l1 && l2){
        if(l1->val < l2->val){
            tail->next = l1;
            l1 = l1->next;
        } else {
            tail->next = l2;
            l2 = l2->next;
        }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
    return dummy.next;
}
```

---

# 🔹 Stack / Queue

## 🔟 Valid Parentheses

```cpp
bool isValid(string s) {
    stack<char> st;
    for(char c : s){
        if(c=='('||c=='{'||c=='[') st.push(c);
        else{
            if(st.empty()) return false;
            char top = st.top(); st.pop();
            if((c==')' && top!='(') ||
               (c=='}' && top!='{') ||
               (c==']' && top!='[')) return false;
        }
    }
    return st.empty();
}
```

---

## 1️⃣1️⃣ Min Stack

```cpp
class MinStack {
    stack<int> s, minS;
public:
    void push(int val) {
        s.push(val);
        if(minS.empty() || val <= minS.top())
            minS.push(val);
    }
    void pop() {
        if(s.top() == minS.top())
            minS.pop();
        s.pop();
    }
    int top() { return s.top(); }
    int getMin() { return minS.top(); }
};
```

---

# 🔹 Trees & Graphs

(Assume TreeNode definition)

```cpp
struct TreeNode {
    int val;
    TreeNode *left, *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

## 1️⃣2️⃣ Binary Tree Level Order

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if(!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while(!q.empty()){
        int size = q.size();
        vector<int> level;
        while(size--){
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if(node->left) q.push(node->left);
            if(node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

---

## 1️⃣3️⃣ Number of Islands (DFS)

```cpp
void dfs(vector<vector<char>>& grid, int i, int j){
    if(i<0 || j<0 || i>=grid.size() || j>=grid[0].size() || grid[i][j]=='0')
        return;
    grid[i][j] = '0';
    dfs(grid,i+1,j);
    dfs(grid,i-1,j);
    dfs(grid,i,j+1);
    dfs(grid,i,j-1);
}

int numIslands(vector<vector<char>>& grid) {
    int count=0;
    for(int i=0;i<grid.size();i++)
        for(int j=0;j<grid[0].size();j++)
            if(grid[i][j]=='1'){
                dfs(grid,i,j);
                count++;
            }
    return count;
}
```

---

# 🔹 Dynamic Programming

## 1️⃣4️⃣ Climbing Stairs

```cpp
int climbStairs(int n) {
    if(n<=2) return n;
    int a=1,b=2;
    for(int i=3;i<=n;i++){
        int c=a+b;
        a=b;
        b=c;
    }
    return b;
}
```

---

## 1️⃣5️⃣ Coin Change

```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount+1, amount+1);
    dp[0]=0;
    for(int i=1;i<=amount;i++)
        for(int coin: coins)
            if(i>=coin)
                dp[i]=min(dp[i], dp[i-coin]+1);
    return dp[amount]>amount?-1:dp[amount];
}
```

---

## 1️⃣6️⃣ Longest Increasing Subsequence (O(n log n))

```cpp
int lengthOfLIS(vector<int>& nums) {
    vector<int> dp;
    for(int num : nums){
        auto it = lower_bound(dp.begin(), dp.end(), num);
        if(it == dp.end()) dp.push_back(num);
        else *it = num;
    }
    return dp.size();
}
```

---

If you want, I can now:

* 🔥 Add **time & space complexity table**
* 🧠 Give **mock interview questions with edge cases**
* 💻 Provide a **complete compilable C++ file**
* 🎯 Focus only on most likely Assurant questions**

Tell me your experience level and I’ll tailor it.



# 用到的方法

```js
Math.max();Math.abs();Math.trunc();Math.floor();Math.ceil();
slice,indexOf,split,join,includes,sort,splice
Set,Map
```



# 高频

```js
螺旋矩阵
var generateMatrix = function (n) {
  let num = 1,
    topRow = 0,
    bottomRow = n - 1,
    leftCol = 0,
    rightCol = n - 1,
    arr = new Array(n).fill(0);
  arr = arr.map(() => new Array(n).fill(0));
  while (num <= Math.pow(n, 2)) {
    for (let i = leftCol; i <= rightCol; i++) arr[topRow][i] = num++;
    topRow++;
    console.log(arr);
    for (let i = topRow; i <= bottomRow; i++) arr[i][rightCol] = num++;
    rightCol--;
    console.log(arr);
    for (let i = rightCol; i >= leftCol; i--) arr[bottomRow][i] = num++;
    bottomRow--;
    console.log(arr);
    for (let i = bottomRow; i >= topRow; i--) arr[i][leftCol] = num++;
    leftCol++;
    console.log(arr);
  }
  return arr;
};
```

两数之和系列

```js
1 两数之和  哈希法(因为要求返回下标，且不用去重)
var twoSum = function (nums, target) {
  let map = new Map();
  for (let i = 0; i < nums.length; i++) {
    let t = target - nums[i];
    if (map.has(t)) {
      return [i, map.get(t)];
    }
    map.set(nums[i], i);
  }
  return [];
};
454 四数相加||  哈希法
var fourSumCount = function (nums1, nums2, nums3, nums4) {
  const map = new Map();
  let count = 0;
  for (const n1 of nums1) {
    for (const n2 of nums2) {
      const sum = n1 + n2;
      if (map.has(sum)) map.set(sum, map.get(sum) + 1);
      else map.set(sum, 1);
    }
  }
  for (const n3 of nums3) {
    for (const n4 of nums4) {
      const sum = n3 + n4;
      if (map.has(0 - sum)) count += map.get(0 - sum);
    }
  }
  return count;
};
三数之和 指针法（因为要给出具体的结果，且去重）
var threeSum = function (nums) {
  let res = [];
  nums.sort((a, b) => a - b); // 先排序
  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue; // 去重i
    let j = i + 1,
      k = nums.length - 1;
    while (j < k) {
      let sum = nums[i] + nums[j] + nums[k];
      if (sum === 0) {
        res.push([nums[i], nums[j], nums[k]]);
        j++;
        k--;
        while (nums[j] === nums[j - 1]) j++; // 去重j
        while (nums[k] === nums[k + 1]) k--; // 去重k
      } else if (sum > 0) {
        k--;
      } else {
        j++;
      }
    }
  }
  return res;
};
四数之和
var fourSum = function (nums, target) {
  const len = nums.length;
  nums.sort((a, b) => a - b);
  const res = [];
  for (let i = 0; i < len - 3; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue; // 去重i
    for (let j = i + 1; j < len - 2; j++) {
      if (j > i + 1 && nums[j] === nums[j - 1]) continue; // 去重j
      let l = j + 1,
        r = len - 1;
      while (l < r) {
        const sum = nums[i] + nums[j] + nums[l] + nums[r];
        if (sum < target) {
          l++;
        } else if (sum > target) {
          r--;
        } else {
          res.push([nums[i], nums[j], nums[l], nums[r]]);
          l++;
          r--;
          while (nums[l] === nums[l - 1]) l++; //去重
          while (nums[r] === nums[r + 1]) r--; //去重
        }
      }
    }
  }
  return res;
};
```

```js
// KMP 时间复杂度 O(m+n)
var strStr = function (haystack, needle) {
  function getNext(arr) {
    //错开一位后，自己匹配自己（这相当于是用前缀去匹配后缀）
    const next = [0];
    for (let i = 1, j = 0; i < arr.length; i++) {
      while (j !== 0 && arr[i] !== arr[j]) j = next[j - 1];
      if (arr[i] === arr[j]) j++;
      next[i] = j;
    }
    return next;
  }
  const next = getNext(needle);
  for (let i = 0, j = 0; i < haystack.length; i++) {
    while (j !== 0 && haystack[i] !== needle[j]) j = next[j - 1];
    if (haystack[i] === needle[j]) j++;
    if (j === needle.length) return i - j + 1;
  }
  return -1;
};
```

```js
// 前k个高频元素
核心是堆排序
var topKFrequent = function (nums, k) {
  const map = new Map(),
    mapArray = [],
    res = [];
  for (let item of nums) {
    map.set(item, map.get(item) ? map.get(item) + 1 : 1);
  }
  for (let item of map) {
    mapArray.push(item);
  }
  createHeap(mapArray);
  let lastIndex = mapArray.length - 1;
  while (k--) {
    res.push(mapArray[0][0]);
    let temp = mapArray[0];
    mapArray[0] = mapArray[lastIndex];
    mapArray[lastIndex] = temp;
    lastIndex--;
    adjust(mapArray, 0, lastIndex);
  }
  return res;
};
function createHeap(map) {
  for (let i = Math.floor(map.length / 2) - 1; i >= 0; i--) {
    adjust(map, i, map.length - 1);
  }
}
function adjust(map, target, lastIndex) {
  for (let i = 2 * target + 1; i <= lastIndex; i = 2 * target + 1) {
    if (i + 1 <= lastIndex && map[i + 1][1] > map[i][1]) i = i + 1;
    if (map[i][1] > map[target][1]) {
      let temp = map[i];
      map[i] = map[target];
      map[target] = temp;
      target = i;
    } else {
      break;
    }
  }
}
```

```js
// 滑动窗口的最大值
单调队列（双向） 存储的是下标！
var maxSlidingWindow = function (nums, k) {
  let q = [],
    res = [];
  for (let i = 0; i < k; i++) {
    while (q.length && nums[q[q.length - 1]] <= nums[i]) q.pop();
    q.push(i);
  }
  res.push(nums[q[0]]);
  for (let i = k; i < nums.length; i++) {
    if (q[0] === i - k) q.shift();
    while (q.length && nums[q[q.length - 1]] <= nums[i]) q.pop();
    q.push(i);
    res.push(nums[q[0]]);
  }
  return res;
};
```



# 双指针

```js
27 移除元素
快指针遍历数组 慢指针指向新数组末尾
var removeElement = function (nums, val) {
  let slow = 0,
    fast = 0;
  for (; fast < nums.length; fast++) {
    if (nums[fast] !== val) {
      nums[slow] = nums[fast];
      slow++;
    }
  }
  return slow;
};
```

```js
长度最小的子数组
var minSubArrayLen = function (target, nums) {
  let len = Infinity,
    slow = 0,
    fast = 0,
    sum = 0;
  for (; fast < nums.length; fast++) {
    sum += nums[fast];
    while (sum >= target) {
      len = fast - slow + 1 < len ? fast - slow + 1 : len;
      sum -= nums[slow];
      slow++;
    }
  }
  return len === Infinity ? 0 : len;
};
```

```js
翻转链表
var reverseList = function (head) {
  let cur = head,
    pre = null;
  while (cur) {
    let temp = cur.next;
    cur.next = pre;
    pre = cur;
    cur = temp;
  }
  return pre;
};
```

```js
两两交换链表的节点
var swapPairs = function (head) {
  let returnHead;
  if (!head) return head;
  if (head.next) {
    returnHead = head.next;
  } else {
    return head;
  }
  let pre = null,
    cur = head;
  while (cur && cur.next) {
    let temp = cur.next.next;
    if (pre) pre.next = cur.next;
    cur.next.next = cur;
    cur.next = temp;
    pre = cur;
    cur = cur.next;
  }
  return returnHead;
};
```

```js
删除链表倒数第N个节点
var removeNthFromEnd = function (head, n) {
  let realHead = new ListNode(0, head),// 删除节点要加头指针！
    fast = head, // 快指针多走一步，方便操作
    slow = realHead;
  while (n--) {
    fast = fast.next;
  }
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return realHead.next;
};
```

```js
链表相交
var getIntersectionNode = function (headA, headB) {
  let h1 = headA,
    h2 = headB;
  while (h1 !== h2) {
    h1 = h1 ? h1.next : headB;
    h2 = h2 ? h2.next : headA;
  }
  return h1;
};
```



# 二叉树

## 遍历

二叉树的遍历实际分为两步：访问与处理

```js
// 拿中序遍历举例
function dfs(root) {
  if (root === null) return; // 访问
  dfs(root.left); // 遍历
  res.push(root.val); // 处理
  dfs(root.right); // 遍历
}
```



## 深度优先 迭代法

用栈模拟递归，且空节点不入栈

```js
// 前序遍历
var preorderTraversal = function (root) {
  let stack = [],
    result = [];
  if (root) stack = [root];
  else return result;
  while (stack.length !== 0) { // 终止条件
    let cur = stack.pop();
    result.push(cur.val);
    cur.right && stack.push(cur.right); // 右节点先进才能后出
    cur.left && stack.push(cur.left);
  }
  return result;
};
```

```js
// 中序遍历
var inorderTraversal = function (root) {
  let cur,
    stack = [],
    res = [];
  if (root) cur = root;
  else return [];
  while (stack.length || cur) { // 终止条件！
    if (cur) { // cur 不为空 则没到最左边
      stack.push(cur); // 访问过的节点存到栈，等会处理
      cur = cur.left;
    } else { // cur 为空 已经到了最左边
      cur = stack.pop(); // 处理逻辑
      res.push(cur.val); // 中 因为已经到最左边了
      cur = cur.right; // 右
    }
  }
  return res;
};
```

```js
// 后序遍历
var postorderTraversal = function (root) {
  let stack = [],
    res = [];
  if (!root) return [];
  stack = [root];
  while (stack.length) {
    let cur = stack.pop();
    res.push(cur.val);
    cur.left && stack.push(cur.left); // 与前序遍历调换位置
    cur.right && stack.push(cur.right);
  }
  return res.reverse(); // 再翻转就是正确的顺序
};
```



## 翻转二叉树

只需要让每个节点的左右孩子调换顺序即可

```js
var invertTree = function (root) {
  if (root === null) return null;
  function inv(node) {
    if (!node) return;
    [node.left, node.right] = [node.right, node.left];
    inv(node.left);
    inv(node.right);
  }
  inv(root);
  return root;
};
```



## 完全二叉树的节点个数

在完全二叉树中，如果递归向左遍历的深度等于递归向右遍历的深度，那说明就是满二叉树

```js
var countNodes = function (root) {
  if (root === null) {
    return 0;
  }
  // 判断是不是满二叉树
  let left = root.left;
  let right = root.right;
  let leftDepth = 0,
    rightDepth = 0;
  while (left) {
    left = left.left;
    leftDepth++;
  }
  while (right) {
    right = right.right;
    rightDepth++;
  }
  if (leftDepth == rightDepth) {
    return Math.pow(2, leftDepth + 1) - 1;
  }
    
  return countNodes(root.left) + countNodes(root.right) + 1;
};
```



## 二叉搜索树

遇到二叉搜索树就用中序遍历转换成有序数组



## 二叉树的最近公共祖先

普通二叉树

```js
var lowestCommonAncestor = function (root, p, q) {
  function dfs(node) {
    if (!node) return null;
    if (node === p || node === q) return node;
    let l = dfs(node.left);
    let r = dfs(node.right);
    if (l && r) return node; // 两个节点在两边
    if (l) return l; // 在一边
    return r;        // 在一边
  }
  return dfs(root);
};
```

二叉搜索树

```js
var lowestCommonAncestor = function (root, p, q) {
  function dfs(node) {
    if (!node) return null;
    if (node.val > p.val && node.val > q.val) return dfs(node.left);
    if (node.val < p.val && node.val < q.val) return dfs(node.right);
    return node;
  }
  return dfs(root);
};
```



## 二叉搜索树的插入删除

```js
var insertIntoBST = function (root, val) {
  function dfs(node) {
    if (!node) return new TreeNode(val); // 找到插入的位置了
    if (node.val > val) node.left = dfs(node.left);
    if (node.val < val) node.right = dfs(node.right);
    return node;
  }
  return dfs(root);
};
```

删除节点

```js
var deleteNode = function (root, key) {
  function dfs(node) {
    if (!node) return null; // 没找到
    if (node.val > key) {
      node.left = dfs(node.left);
      return node;
    } else if (node.val < key) {
      node.right = dfs(node.right);
      return node;
    } else {
      // 找到了
      if (!node.left) return node.right;
      else if (!node.right) return node.left;
      else {
        //左右孩子都不为空，则将删除节点的左孩子
        //放到删除节点的右子树的最左面节点的左孩子上，
        //返回删除节点右孩子为新的根节点
        let cur = node.right;
        while (cur.left) {
          cur = cur.left;
        }
        cur.left = node.left;
        return node.right;
      }
    }
  }
  return dfs(root);
};
```



# 回溯

## 总结

回溯是穷举，抽象为树结构。回溯的优化只能剪枝。

- 收集叶子还是每个节点（子集问题）,全部收集的逻辑写的位置变一下
- 需要 start 吗？
- 只有排列问题需要树枝去重，在递归函数外面定义一个 used 数组即可
- 树层去重（数组有重复元素，但结果不能有重复）
  - 要么先排序，再在 for 循环判断是否和前一个相等
  - 不能排序的话，在 for 循环里面定义 used 数组
- 树枝树层都要去重：看全排列||

```js
function backtracking(参数) {
    if (终止条件) {
        // 存放结果
        return;
    }
    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        // 回溯
    }
}
```



## 组合问题

```js
// 组合总数||
// 数组有重复元素，但还不能有重复的组合:树层去重
var combinationSum2 = function (candidates, target) {
  candidates = candidates.sort((a, b) => a - b); // 树层去重必须先排序
  function track(sum, start) {
    if (sum === target) {
      res.push(path.slice());
      return;
    } else if (sum > target) {
      return;
    }
    for (let i = start; i < candidates.length; i++) { // for循环就是一层
      if (i - 1 >= start && candidates[i] === candidates[i - 1]) continue;
      // 数层去重,前面一个相等说明用过了
      path.push(candidates[i]);
      track(sum + candidates[i], i + 1);
      path.pop();
    }
  }
  let res = [],
    path = [];
  track(0, 0);
  return res;
};
```



## 分割问题

分割也是组合问题 每次决定切哪一部分下来

```js
// 分割回文串

var partition = function (s) {
  function is(s) {
    for (let i = 0; i < s.length; i++) {
      if (s[i] === s[s.length - i - 1]) {
        continue;
      } else {
        return false;
      }
    }
    return true;
  }
  let path = [],
    res = [];
  function track(start) {
    if (start === s.length) {
      res.push([...path]);
      return;
    }
    for (let i = start; i < s.length; i++) {
      let str = s.slice(start, i + 1);
      if (is(str)) {
        path.push(str);
        track(i + 1);
        path.pop();
      } else {
        continue;
      }
    }
  }
  track(0);
  return res;
};
```



## 排列问题

所有排列问题都要树枝去重

```js
var permute = function (nums) {
  let res = [],
    path = [],
    used = new Array(nums.length).fill(false);// 树枝去重，path里不能重复选择
  track();
  return res;
  function track() {
    if (path.length === nums.length) {
      res.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      used[i] = true;
      path.push(nums[i]);
      track();
      path.pop();
      used[i] = false;
    }
  }
};
```

```js
// 全排列||
// 树枝+树层去重
var permuteUnique = function (nums) {
  nums.sort((a, b) => a - b);// 树层去重 先排列
  let res = [],
    path = [],
    used = new Array(nums.length).fill(false);//给树枝用
  track();
  return res;
  function track() {
    if (path.length === nums.length) {
      res.push(path.slice());
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i] === true) continue;// 树枝去重就一句话
      if (i > 0 && nums[i - 1] === nums[i] && !used[i - 1]) continue;
      // !used[i - 1] 注意！！
      path.push(nums[i]);
      used[i] = true;
      track();
      path.pop();
      used[i] = false;
    }
  }
};
```



## 子集问题

子集也是一种组合问题，组合问题都是无序的（决定 for 从哪开始）

组合问题和分割问题都是收集树的叶子节点，而子集问题是找树的所有节点

```js
var subsets = function (nums) {
  let res = [[]],// 空[]也要进去
    path = [];
  track(0);
  return res;

  function track(start) {
    if (start === nums.length) return;
    for (let i = start; i < nums.length; i++) {
      path.push(nums[i]);
      res.push([...path]); // 子集问题收集每个节点
      track(i + 1);
      path.pop();
    }
  }
};
```

```js
// 结果里不允许有重复：树层去重
var subsetsWithDup = function (nums) {
  nums.sort((a, b) => a - b); // 先排序
  let res = [[]],// 空[]也要进去
    path = [];
  track(0);
  return res;
  function track(start) {
    if (start === nums.length) return;
    for (let i = start; i < nums.length; i++) {
      if (nums[i - 1] === nums[i] && i > start) continue; // 树层去重，和组合里一个道理
      path.push(nums[i]);
      res.push(path.slice()); // 子集问题收集每个节点
      track(i + 1);
      path.pop();
    }
  }
};
```

```js
// 递增子序列
var findSubsequences = function (nums) {
  let res = [],
    path = [];
  track(0);
  return res;
  function track(start) {
    if (start === nums.length) {
      return;
    }
    let used = []; // 数层去重第二种方式（因为不能排序了）
    for (let i = start; i < nums.length; i++) {
      if (used.includes(nums[i]) || nums[i] < path[path.length - 1]) continue;
      path.push(nums[i]);
      used.push(nums[i]);
      if (path.length >= 2) res.push(path.slice());
      track(i + 1, []);
      path.pop();
    }
  }
};
```



## 棋盘问题

N 皇后



# 动态规划

动态规划是由前一个状态推导出来的，而贪心是局部直接选最优的。

1. dp数组定义
2. 确定递推公式
3. dp数组如何初始化
4. 确定遍历顺序



## 基础题

```js
// 343整数拆分
var integerBreak = function (n) {
  const dp = [0, 1, 1];
  for (let i = 3; i <= n; i++) {
    for (let j = 1; j <= i; j++) {
      dp[i] = Math.max(j * (i - j), dp[j] * (i - j), dp[i] ?? 0);
    }
  }
  return dp[n];
};
```



## 背包

0-1背包：**每个物品选与不选的问题**

从后向前遍历

完全背包：**每个物品可以选多次**

从前向后遍历

```js
// 递推公式：两者最值、相加
dp[j] = Math.max(dp[j - weight[i]] + value[i], dp[j])// 初始化 dp[0] = 0
dp[j] += dp[j - weight[i]]// 初始化 dp[0] = 1
dp[j] = dp[j - weight[i]] || dp[j]// 初始化 dp[0] = true
```

如果求组合数，外层for循环遍历物品，内层for遍历背包。

如果求排列数（有顺序），外层for遍历背包，内层for循环遍历物品。



```js
// 416分割等和子集
var canPartition = function (nums) {
  let sum = nums.reduce((pre, val) => pre + val, 0);
  sum = sum / 2;
  if (sum !== Math.floor(sum)) return false;
  const dp = new Array(sum + 1).fill(0);
  for (let i = 1; i <= sum; i++) if (i >= nums[0]) dp[i] = nums[0];
  for (let i = 1; i < nums.length; i++)
    for (let j = sum; j > nums[i]; j--) {
      dp[j] = Math.max(dp[j - nums[i]] + nums[i], dp[j]);
    }
  return dp[sum] === sum;
};
```

```js
// 139.单词拆分
var wordBreak = function (s, wordDict) {
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;
  for (let j = 1; j <= s.length; j++)
    for (let i = 0; i < wordDict.length; i++)
      dp[j] =
        dp[j] ||
        (j >= wordDict[i].length &&
          dp[j - wordDict[i].length] &&
          wordDict[i] === s.slice(j - wordDict[i].length, j));
  return dp[s.length];
};
```



## 打家劫舍

```js
//2的代码
var rob = function (nums) {
  if (nums.length == 1) return nums[0];
  return Math.max(robb(nums.slice(0, nums.length - 1)), robb(nums.slice(1)));
};
function robb(arr) {
  //1的代码
  if (arr.length === 1) return arr[0];
  let dp = new Array(arr.length - 1).fill(0);
  dp[0] = arr[0];
  dp[1] = Math.max(arr[0], arr[1]);
  for (let i = 2; i < arr.length; i++)
    dp[i] = Math.max(dp[i - 1], dp[i - 2] + arr[i]);
  return dp[dp.length - 1];
}
```

```js
// 打家劫舍3 树形dp 后序遍历
var rob = function (root) {
  function dfs(node) {
    if (node === null) return [0, 0];
    let yes, no;
    let left = dfs(node.left);
    let right = dfs(node.right);
    yes = left[1] + right[1] + node.val;
    no = Math.max(...left) + Math.max(...right);
    return [yes, no];
  }
  return Math.max(...dfs(root));
};
```



## 股票问题

股票问题公式：

```js
// 注：k=0时 dp=0 因为没有交易所以获利为零
// 通用
dp[i][k][0] = max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i]);
dp[i][k][1] = max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i]);
// k=1
dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
dp[i][1] = Math.max(dp[i - 1][1], -prices[i]);
// k=2
dp[i][2][0] = Math.max(dp[i - 1][2][0], dp[i - 1][2][1] + prices[i]);
dp[i][2][1] = Math.max(dp[i - 1][2][1], dp[i - 1][1][0] - prices[i]);
dp[i][1][0] = Math.max(dp[i - 1][1][0], dp[i - 1][1][1] + prices[i]);
dp[i][1][1] = Math.max(dp[i - 1][1][1], -prices[i]);
// k为正无穷
dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
// k为正无穷 冷冻期1天
dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
dp[i][1] = max(dp[i - 1][1], dp[i - 2][0] - prices[i]);
// k为正无穷 有手续费 只能卖出时扣手续费！！！
dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i] - fee);

```



```js
// 通解 k为任意值
var maxProfit = function (k, prices) {
  if (k > prices.length / 2) return infinity(prices);
  //一个有收益的交易至少需要两天（在前一天买入，在后一天卖出，前提是买入价格低于卖出价格）
  //如果股票价格数组的长度为 n，则有收益的交易的数量最多为 n / 2（整数除法）
  //因此 k 的临界值是 n / 2。如果给定的 k 不小于临界值，即 k >= n / 2
  //则可以将 k 扩展为正无穷，此时问题等价于 k 为正无穷
  const dp = new Array(prices.length)
    .fill(0)
    .map(() => new Array(k + 1).fill(0).map(() => new Array(2).fill(0)));
  // 通解时，必须 fill 0，不能 infinity
  for (let j = 1; j <= k; j++) {
    dp[0][j][0] = 0;
    dp[0][j][1] = -prices[0];
  }
  for (let i = 1; i < prices.length; i++)
    for (let j = 1; j <= k; j++) {
      dp[i][j][0] = Math.max(dp[i - 1][j][0], dp[i - 1][j][1] + prices[i]);
      dp[i][j][1] = Math.max(dp[i - 1][j][1], dp[i - 1][j - 1][0] - prices[i]);
    }
  return dp[dp.length - 1][k][0];
};
function infinity(prices) {
  const dp = new Array(prices.length)
    .fill(0)
    .map(() => new Array(2).fill(-Infinity));
  (dp[0][0] = 0), (dp[0][1] = -prices[0]);
  for (let i = 1; i < prices.length; i++) {
    dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
  }
  return dp[dp.length - 1][0];
}
```

```js
// 含冷冻期
var maxProfit = function (prices) {
  const dp = new Array(prices.length)
    .fill(0)
    .map(() => new Array(2).fill(-Infinity));
  dp[0][0] = 0;
  dp[0][1] = -prices[0];
  for (let i = 1; i < prices.length; i++) {
    dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i - 1][1], (i >= 2 ? dp[i - 2][0] : 0) - prices[i]);
  }
  return dp[dp.length - 1][0];
};
```

```js
// 含手续费
var maxProfit = function (prices, fee) {
  const dp = new Array(prices.length)
    .fill(0)
    .map(() => new Array(2).fill(-Infinity));
  dp[0][0] = 0;
  dp[0][1] = -prices[0] - fee; // 减fee
  for (let i = 1; i < prices.length; i++) {
    dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i] - fee); // 公式只能是这个
  }
  return dp[dp.length - 1][0];
};
```



## 子序列问题



### 连续与不连续

子串、子数组：无论一维还是二维，dp 定义指的都是以 i 结尾的

子序列：一维是 dp 以 i 结尾，二维是到 i 的子序列



### 编辑距离

编辑距离问题为了方便 dp i j 指的是 i-1 j-1

```js
// 115.不同的子序列
var numDistinct = function (s, t) {
  let dp = new Array(s.length + 1)
    .fill(0)
    .map((_) => new Array(t.length + 1).fill(0));
  for (let i = 0; i <= s.length; i++) dp[i][0] = 1;
  for (let i = 1; i <= s.length; i++)
    for (let j = 1; j <= t.length; j++) {
      if (s[i - 1] === t[j - 1]) dp[i][j] = dp[i - 1][j] + dp[i - 1][j - 1];
      else dp[i][j] = dp[i - 1][j];
    }
  return dp[s.length][t.length];
};
```

```js
// 72. 编辑距离
```



### 回文

```js
// 647 回文子串
// 因为是子串 所以dp[i][j] 定义为 i-j是不是回文子串（boolean）
var countSubstrings = function (s) {
  let dp = new Array(s.length)
      .fill(0)
      .map(() => new Array(s.length).fill(false)),
    res = 0;

  for (let i = s.length - 1; i >= 0; i--)
    for (let j = i; j < s.length; j++)
      if (s[i] === s[j] && (j - i <= 1 || dp[i + 1][j - 1])) {//递推公式 j-i<=1 写前面
        dp[i][j] = true;
        res++;
      }
  return res;
};
```

```js
// 516 最长回文子序列
// 回文子序列 dp定义为 i-j字符串 子序列长度
var longestPalindromeSubseq = function (s) {
  let dp = new Array(s.length).fill(0).map(() => new Array(s.length).fill(1));
  for (let i = s.length - 1; i >= 0; i--)
    for (let j = i + 1; j < s.length; j++) {
      if (s[i] == s[j]) {
        if (j == i + 1) dp[i][j] = 2;
        else dp[i][j] = dp[i + 1][j - 1] + 2;
      } else {
        dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
      }
    }
  return dp[0][s.length - 1];
};
```



# 贪心

区间问题套路：贪心, **先排序, 然后遍历检查是否满足合并区间的条件**。



# 单调栈

通常是一维数组，要寻找任一个元素的右边或者左边第一个比自己大或者小的元素的位置，此时我们就要想到可以用单调栈了。

Binary Search is a fast searching algorithm that works on **sorted arrays**. Instead of checking every element one by one, it cuts the search space in half each time — making it much faster than Linear Search for large arrays.

**Prerequisite:** The array must be sorted before using Binary Search.

---

## How It Works (Simple Explanation)

Think of looking up a word in a dictionary. You don't start from page 1 — you open the book in the middle. If your word comes before the middle word, you search the left half. If it comes after, you search the right half. You keep halving until you find it.

Binary Search does exactly the same thing with an array.

---

## Step-by-Step Walkthrough

Array: `[ 1, 3, 5, 7, 9, 11, 13 ]` — searching for **7**

```
low = 0, high = 6
mid = (0 + 6) / 2 = 3  →  arr[3] = 7  ✓ Found at index 3!
```

Now searching for **11**:

```
low = 0, high = 6,  mid = 3  →  arr[3] = 7  →  11 > 7, search right half
low = 4, high = 6,  mid = 5  →  arr[5] = 11 ✓ Found at index 5!
```

Now searching for **4** (not in array):

```
low = 0, high = 6,  mid = 3  →  arr[3] = 7  →  4 < 7, search left half
low = 0, high = 2,  mid = 1  →  arr[1] = 3  →  4 > 3, search right half
low = 2, high = 2,  mid = 2  →  arr[2] = 5  →  4 < 5, search left half
low = 2, high = 1  →  low > high → Not found, return -1
```

---

## Java Example

### Iterative approach

```java
public class BinarySearch {

    static int binarySearch(int[] arr, int target) {
        int low  = 0;
        int high = arr.length - 1;

        while (low <= high) {
            int mid = (low + high) / 2;

            if (arr[mid] == target) {
                return mid;          // found — return the index
            } else if (arr[mid] < target) {
                low = mid + 1;       // target is in the right half
            } else {
                high = mid - 1;      // target is in the left half
            }
        }

        return -1; // not found
    }

    public static void main(String[] args) {
        int[] numbers = {1, 3, 5, 7, 9, 11, 13};

        System.out.println(binarySearch(numbers, 7));  // Output: 3
        System.out.println(binarySearch(numbers, 11)); // Output: 5
        System.out.println(binarySearch(numbers, 4));  // Output: -1
    }
}
```

---

## Binary Search vs Linear Search

| | Binary Search | Linear Search |
|---|---|---|
| Array must be sorted | Yes | No |
| Best case | O(1) | O(1) |
| Worst case | O(log n) | O(n) |
| Strategy | Divide and conquer | Check one by one |
| Best for | Large sorted arrays | Small or unsorted arrays |

---

## Time & Space Complexity

| Case | Complexity | Reason |
|---|---|---|
| Best Case | O(1) | Target is the middle element on first check |
| Worst Case | O(log n) | Search space is halved each step |
| Space (iterative) | O(1) | No extra memory needed |

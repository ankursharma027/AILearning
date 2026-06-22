Linear Search, also called Sequential Search, goes through an array from the first element to the last, comparing each element with the target. It stops as soon as the target is found or the end of the array is reached.

It works on both **sorted and unsorted** arrays — no preparation needed.

---

## How It Works (Simple Explanation)

Think of looking for a specific book on a shelf by checking each book one by one from left to right until you find the right one (or run out of books).

---

## Step-by-Step Walkthrough

Array: `[ 4, 2, 9, 7, 5 ]` — searching for **7**

```
Check index 0 → 4 ≠ 7, continue
Check index 1 → 2 ≠ 7, continue
Check index 2 → 9 ≠ 7, continue
Check index 3 → 7 = 7 ✓ Found at index 3!
```

Searching for **6** (not in array):

```
Check index 0 → 4 ≠ 6
Check index 1 → 2 ≠ 6
Check index 2 → 9 ≠ 6
Check index 3 → 7 ≠ 6
Check index 4 → 5 ≠ 6
End of array → return -1 (not found)
```

---

## Java Example

```java
public class LinearSearch {

    public int searchData(int data, int[] array) {
        int index = -1;
        for (int i = 0; i < array.length; i++) {
            if (array[i] == data) {
                index = i;
                break;
            }
        }
        return index;
    }

    public static void main(String[] args) {
        LinearSearch ls = new LinearSearch();
        int[] numbers = {4, 2, 9, 7, 5};

        System.out.println(ls.searchData(7, numbers)); // Output: 3
        System.out.println(ls.searchData(6, numbers)); // Output: -1
    }
}
```

---

## Time & Space Complexity

| Case | Complexity | Reason |
|---|---|---|
| Best Case | O(1) | Target is the first element |
| Average Case | O(n) | Target is somewhere in the middle |
| Worst Case | O(n) | Target is the last element or not present |
| Space | O(1) | No extra memory used |

Bubble sort works by repeatedly stepping through the list, comparing adjacent elements, and swapping them if they are in the wrong order. This process repeats until the list is sorted, with the largest elements "bubbling" to their correct positions at the end of the array during each pass.

---

## How It Works (Simple Explanation)

Imagine you have a row of numbers. You walk through the row, look at two neighbours at a time, and swap them if the left one is bigger. After one full walk, the largest number has "bubbled up" to the last position. You repeat this for the remaining unsorted part.

---

## Step-by-Step Walkthrough

Array: `[ 5, 3, 8, 1 ]`

**Pass 1:**
- Compare 5 and 3 → swap → `[ 3, 5, 8, 1 ]`
- Compare 5 and 8 → no swap → `[ 3, 5, 8, 1 ]`
- Compare 8 and 1 → swap → `[ 3, 5, 1, 8 ]`  ← 8 is in its final place

**Pass 2:**
- Compare 3 and 5 → no swap → `[ 3, 5, 1, 8 ]`
- Compare 5 and 1 → swap → `[ 3, 1, 5, 8 ]`  ← 5 is in its final place

**Pass 3:**
- Compare 3 and 1 → swap → `[ 1, 3, 5, 8 ]`  ← sorted!

Each pass pushes the next largest element to its correct spot at the end.

---

## Java Example

```java
public class BubbleSort {

    public int[] sortData(int[] array) {

        for (int i = 0; i < array.length; i++) {
            for (int j = i + 1; j < array.length; j++) {
                if (array[i] > array[j]) {
                    int temp;
                    temp = array[i];
                    array[i] = array[j];
                    array[j] = temp;
                }
            }
        }
        return array;
    }

    public static void main(String[] args) {
        int[] numbers = {5, 3, 8, 1};
        bubbleSort(numbers);

        for (int n : numbers) {
            System.out.print(n + " "); // Output: 1 3 5 8
        }
    }
}
```
---

## Time & Space Complexity

| Case | Time Complexity | When |
|---|---|---|
| Best Case | O(n) | Array is already sorted (with `swapped` flag) |
| Average Case | O(n²) | Elements are in random order |
| Worst Case | O(n²) | Array is in reverse order |
| Space | O(1) | Sorting is done in-place, no extra array needed |

---

## Key Points

- Simple to understand but **slow on large data** — not used in production for big arrays.
- Sorts **in-place** (no extra memory).
- **Stable sort** — equal elements keep their original relative order.
- Good for learning; in practice, use **Merge Sort** or **Quick Sort** for better performance.

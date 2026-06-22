Quick Sort is a divide-and-conquer sorting algorithm. It picks a **pivot** element, puts all smaller elements to its left and all larger elements to its right, then recursively sorts both sides. On average it is much faster than Bubble Sort.

---

## How It Works (Simple Explanation)

Think of sorting a hand of cards. You pick one card as a reference (the pivot). You move all cards smaller than it to the left and all cards bigger to the right. Now that pivot card is in its final correct position. You then repeat the same process on the left group and the right group independently.

---

## Step-by-Step Walkthrough

Array: `[ 5, 3, 8, 1, 4 ]` — pivot = last element = **4**

**Partition:**
- 5 > 4 → leave
- 3 < 4 → move to left side
- 8 > 4 → leave
- 1 < 4 → move to left side
- Place pivot **4** in its correct spot

Result: `[ 3, 1, 4, 5, 8 ]` ← 4 is now in its final position

**Recurse left side** `[ 3, 1 ]` → pivot = 1
- 3 > 1 → swap → `[ 1, 3 ]` ✓

**Recurse right side** `[ 5, 8 ]` → pivot = 8
- 5 < 8 → already in place → `[ 5, 8 ]` ✓

**Final sorted array:** `[ 1, 3, 4, 5, 8 ]`

---

## Java Example

```java
public class QuickSort {

    static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);
            quickSort(arr, low, pivotIndex - 1);  // sort left side
            quickSort(arr, pivotIndex + 1, high); // sort right side
        }
    }

    static int partition(int[] arr, int low, int high) {
        int pivot = arr[high]; // pick last element as pivot
        int i = low - 1;       // i tracks the boundary of smaller elements

        for (int j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                // swap arr[i] and arr[j]
                int temp = arr[i];
                arr[i]   = arr[j];
                arr[j]   = temp;
            }
        }

        // place pivot in its correct position
        int temp    = arr[i + 1];
        arr[i + 1]  = arr[high];
        arr[high]   = temp;

        return i + 1; // return pivot's final index
    }

    public static void main(String[] args) {
        int[] numbers = {5, 3, 8, 1, 4};
        quickSort(numbers, 0, numbers.length - 1);

        for (int n : numbers) {
            System.out.print(n + " "); // Output: 1 3 4 5 8
        }
    }
}
```

---

## Time & Space Complexity

| Case | Time Complexity | When |
|---|---|---|
| Best Case | O(n log n) | Pivot always splits the array evenly |
| Average Case | O(n log n) | Random data |
| Worst Case | O(n²) | Pivot is always the smallest or largest (sorted/reverse array) |
| Space | O(log n) | Recursive call stack |

---

## Quick Sort vs Bubble Sort

| | Quick Sort | Bubble Sort |
|---|---|---|
| Speed | Fast — O(n log n) average | Slow — O(n²) always |
| Strategy | Divide and conquer | Compare neighbours repeatedly |
| In-place | Yes | Yes |
| Stable | No | Yes |
| Best for | Large datasets | Learning / tiny arrays |


# BÀI THỰC HÀNH SỐ 1

## 1. Mục đích
- Tìm hiểu cách thức lập trình đa luồng.

## 2. Nội dung
- Viết chương trình thực hiện sắp xếp trộn với mảng > 1.000.000.000 phần tử bằng cách chia mảng thành `n` (n là số CPU thực tế trong máy tính) phần dữ liệu với số lượng phần tử mỗi phần tương đương nhau. 
- Mỗi phần dữ liệu sẽ tạo một luồng để thực thi.
- So sánh thời gian thực hiện với trường hợp một luồng. (Sử dụng thư viện luồng của ngôn ngữ lập trình tùy chọn).

---

## 3. Giải pháp lập trình

### **Chuẩn bị môi trường**
- Sử dụng Java.
- Dùng `ForkJoinPool` hoặc `Thread` để tạo đa luồng.
- Đo thời gian thực thi bằng `System.currentTimeMillis()`.

### **Thuật toán triển khai**
1. **Chia mảng**: Tách mảng thành `n` phần nhỏ, với `n = Runtime.getRuntime().availableProcessors()` (số CPU thực tế).
2. **Đa luồng**: Sử dụng một luồng riêng để sắp xếp từng phần tử đã chia.
3. **Gộp kết quả**: Kết hợp các mảng đã sắp xếp bằng thuật toán merge.
4. **So sánh thời gian**: Đo và so sánh hiệu suất giữa đa luồng và đơn luồng.

---

## 4. Mã nguồn (Java)

```java
import java.util.Arrays;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class MultiThreadedMergeSort {

    public static void main(String[] args) throws InterruptedException {
        // Kích thước mảng
        int size = 1_000_000_000;
        int[] array = new int[size];
        for (int i = 0; i < size; i++) {
            array[i] = (int) (Math.random() * size);
        }

        // Số lượng CPU
        int numThreads = Runtime.getRuntime().availableProcessors();

        // Chia mảng
        int[][] subArrays = splitArray(array, numThreads);

        // Đo thời gian
        long startTime = System.currentTimeMillis();

        // Sắp xếp từng phần bằng đa luồng
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        for (int i = 0; i < numThreads; i++) {
            final int index = i;
            executor.submit(() -> Arrays.sort(subArrays[index]));
        }
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.HOURS);

        // Gộp mảng
        int[] sortedArray = mergeSubArrays(subArrays);

        long endTime = System.currentTimeMillis();
        System.out.println("Thời gian thực thi (đa luồng): " + (endTime - startTime) + " ms");

        // So sánh với trường hợp đơn luồng
        startTime = System.currentTimeMillis();
        Arrays.sort(array);
        endTime = System.currentTimeMillis();
        System.out.println("Thời gian thực thi (đơn luồng): " + (endTime - startTime) + " ms");
    }

    // Hàm chia mảng
    public static int[][] splitArray(int[] array, int numParts) {
        int[][] result = new int[numParts][];
        int chunkSize = array.length / numParts;
        for (int i = 0; i < numParts; i++) {
            int start = i * chunkSize;
            int end = (i == numParts - 1) ? array.length : (i + 1) * chunkSize;
            result[i] = Arrays.copyOfRange(array, start, end);
        }
        return result;
    }

    // Hàm gộp mảng
    public static int[] mergeSubArrays(int[][] subArrays) {
        int[] result = subArrays[0];
        for (int i = 1; i < subArrays.length; i++) {
            result = mergeTwoArrays(result, subArrays[i]);
        }
        return result;
    }

    // Gộp hai mảng đã sắp xếp
    public static int[] mergeTwoArrays(int[] arr1, int[] arr2) {
        int[] result = new int[arr1.length + arr2.length];
        int i = 0, j = 0, k = 0;
        while (i < arr1.length && j < arr2.length) {
            result[k++] = (arr1[i] <= arr2[j]) ? arr1[i++] : arr2[j++];
        }
        while (i < arr1.length) {
            result[k++] = arr1[i++];
        }
        while (j < arr2.length) {
            result[k++] = arr2[j++];
        }
        return result;
    }
}

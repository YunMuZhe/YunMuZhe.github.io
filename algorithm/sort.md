注：以下的所有代码组合起来可以作为一个可运行的java类，如果想要拿下来学习的话可以试着测试一下。

```java
/**
 * 几种基础排序算法的实现
 */
package com.edu.hfut;

import java.lang.reflect.Method;
import java.util.Arrays;

/**
   * 排序算法的练习
 * @author hfut-zxq 
 * @e-mail zxq1194792327@163.com
 *
 */
public class Sort {
	public static void main(String[] args) {
		checking("halfInsertSort", "插入排序");
		checking("shellSort", "希尔排序");
		checking("chooseSort", "选择排序");
		checking("bubbleSort", "冒泡排序");
		checking("fastSort", "快速排序");
		checking("mergerSort", "归并排序");
	}
	
	/**
	 * 用反射的模式来验证某个排序是否可以正常运行
	 * @param methodName 要调用的排序算法的方法名
	 * @param nums 测试的
	 * @param sortMethodName 要调用的排序算法的中文名
	 * @notes 数组克隆的四种方法：
	 * @notes 1. int[] nums2 = Arrays.copyOf(nums, nums.length);
	 * @notes 2. int[] nums3 = Arrays.copyOfRange(nums, 0, nums.length - 1);
	 * @notes int[] nums4 = new int[nums.length];
	 * @notes 3. System.arraycopy(nums, 0, nums4, 0, nums.length);
	 * @notes 4. int[] nums5 = (int[])nums.clone();
	 */
	public static void checking(String methodName, String sortMethodName){
		Sort sort = new Sort();
		Class<Sort> bean = Sort.class;
		// 测试次数
		int testNum = 100;
		for(int j = 0; j < testNum; j++) {
			// 此次测试中所用的int[]数组的元素个数（大于等于1，小于等于100）
			int num = (int)(Math.random() * 100 + 1);
			// 定义本次测试放入Arrays.sort()方法中所用的数组并进行随机初始化
			int[] nums = new int[num];
			for(int i = 0; i < num; i++) {
				nums[i] = (int)(Math.random() * 100 + 1);
			}
			// 复制一个放入methodName算法运行的数组
			int[] tmp = Arrays.copyOf(nums, nums.length);
			// 排序
			Arrays.sort(nums);
			try {
				// 得到methodName方法
				Method md = bean.getMethod(methodName, int[].class);
				// 调用此方法
				md.invoke(sort, tmp);
				// 得到判断算法是否成功的方法
				md = bean.getMethod("judgeIsOK", int[].class, int[].class, int.class, int.class);
				// 进行判断
				int i = (int)(md.invoke(sort, nums, tmp, 0, nums.length));
				if(i != -1) {
					System.out.println("i = " + i + 
							"时，" + sortMethodName + "出现错误！");
					System.out.println("正常排序算法此处为： " + nums[i]);
					System.out.println("而 " + sortMethodName + " 在此处为：" + tmp[i]);
					return ;
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		System.out.println(sortMethodName + " 算法 OK！");
	}
	
	/**
	 * 验证排序是否正确
	 * @param sortMethodName 算法名称
	 * @param nums1   使用 Arrays.sort(nums1)排过序后的数组
	 * @param nums2   使用 sortMethodName(nums2) 算法排过序后的数组
	 * @param start 排序开始的index，在比较序列之间
	 * @param end 	排序结束的index，不在比较序列之间
	 */
	public static int judgeIsOK(int[] nums1, int[] nums2, int start, int end) {
		for (int i = start; i < end; i++) {
			if(nums1[i] != nums2[i]) {
				return i;
			}
		}
		return -1;
	}
```

## 插入类排序

### 插入排序第一次练习

```java
	/**
	 * 插入排序第一次练习
	 * 插入排序特点：对小规模/大体上已经排好序的数组效率较高
	 * @param nums
	 */
	public static void insertSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		int length = nums.length;
		int hasSort = 1;
		while(hasSort < length) {
			int tmp = nums[hasSort];
			// 找到比tmp大的数所在的index——i
			int i = 0;
			for(; i < hasSort; i++) {
				if(nums[i] > tmp) {
					break;
				}
			}
			// 从hasSort那个index往前一直到i，每个数都往后移一下
			for(int j = hasSort-1; j >= i; j--) {
				nums[j+1] = nums[j];
			}
			// 将tmp插到i处
			nums[i] = tmp;
			// hasSort + 1
			hasSort += 1;
		}
	}
```


### 插入排序改进版

```java
	/**
	 * 插入排序改进版--插入排序在寻找合适的插入位置时，也可以从后往前找，边找边把元素往后移
	 * @param nums
	 */
	public static void insertSort2(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		int length = nums.length;
		int hasSort = 1;
		while(hasSort < length) {
			int tmp = nums[hasSort];
			int i = hasSort - 1;
			while(i >= 0 && nums[i] > tmp) {
				nums[i+1] = nums[i];
				i--;
			}
			// 此时 i 所在的位置为tmp应该插入的前一个位置
			nums[i+1] = tmp;
			hasSort += 1;
		}
	}
```

### 插入排序的再一次改进：二分折半插入排序
```java
	/**
	     *  插入排序的再一次改进：二分折半插入排序
	     * 【查找合适位置】的实现用上了二分查找
	 */
	public static void halfInsertSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		int hasSort = 1;
		int length = nums.length;
		while(hasSort < length) {
			int tmp = nums[hasSort];
			int left, right, mid;
			left = 0;
			right = hasSort - 1;
			while(left <= right) {
				mid = (left+right) / 2;
				if(nums[mid] == tmp) {
					left = mid;
					break;
				} else if(nums[mid] > tmp) {
					right = mid - 1;
				} else {
					left = mid + 1;
				}
			}
			for(int i = hasSort - 1; i >= left; i--) {
				nums[i+1] = nums[i];
			}
			nums[left] = tmp;
			hasSort ++;
		}
	}
```

### 希尔排序
```java
	/**
	 *  希尔排序，增加了一个gap，index相差gap的元素组成子数组，对子数组进行插入排序，然后再逐渐减小gap，完成排序
	 *  特点：相较于插入排序，对稍微大一些规模的数组，效率更高
	 * @param nums
	 */
	public static void shellSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		int[] gap = new int[] {1, 2, 4, 8};
		for(int i = gap.length - 1; i >= 0; i--) {
			shellSortHelper_insertSort(nums, gap[i]);
		}
	}
	/**
     * 希尔排序的辅助函数，对子序列们进行插入排序
	 * @param nums
	 * @param gap
	 */
	public static void shellSortHelper_insertSort(int[] nums, int gap) {
		int tmp;
		// 进行gap+1次的排序——即对gap+1个子序列进行插入排序
		for(int i = gap; i < nums.length; i++) {
			if(nums[i-gap] > nums[i]) {
				tmp = nums[i];
				int j = i-gap;
				while(j >= 0 && nums[j] > tmp) {
					nums[j+gap] = nums[j];
					j -= gap;
				}
				nums[j+gap] = tmp;
			}
		}
	}
```

## 交换类排序

### 冒泡排序
```java
	/**
	 * 冒泡排序
	 */
	public static void bubbleSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		for(int i = 0; i < nums.length; i++) {
			for(int j = i + 1; j<nums.length; j++) {
				if(nums[j] < nums[i]) {
					int tmp = nums[i];
					nums[i] = nums[j];
					nums[j] = tmp;
				}
			}
		}
	}
```

### 快速排序
```java
	/**
	 * 快速排序
	 * @param nums
	 */
	public static void fastSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		
(nums, 0, nums.length-1);
	}

	/**
	 * 快速排序的辅助方法
	 * @param nums
	 * @param left
	 * @param right
	 */
	public static void fastSortHelper(int[] nums, int left, int right) {
		if(left >= right) {
			return ;
		}
		int start = left, end = right;
		int tmp = nums[left];
		while(start < end) {
			while(nums[end] >= tmp && end > start) {
				end--;
			}
			nums[start] = nums[end];
			while(nums[start] <= tmp && start < end) {
				start++;
			}
			nums[end] = nums[start];
		}
		nums[start] = tmp;
		fastSortHelper(nums, left, start-1);
		fastSortHelper(nums, end+1, right);
	}
```

## 归并排序

### 归并排序

```java
	/**
	   *  归并排序
	 * @param nums
	 */
	public static void mergerSort(int[] nums) {
		if(nums == null || nums.length < 2) {
			return ;
		}
		mergerSortHelper1(nums, 0, nums.length - 1);
	}

	public static void mergerSortHelper1(int[] nums, int left, int right) {
		if(left < right) {
			mergerSortHelper1(nums, left, (left + right) / 2);
			mergerSortHelper1(nums, (left + right) / 2 + 1, right);
			mergerSortHelper2(nums, left, right);
		}
	}

	public static void mergerSortHelper2(int[] nums, int left, int right) {
		if(left == right) return ;
		int i = left, j = (left + right) / 2 + 1;
		int[] numsTmp = nums.clone();
		int key = left;
		while(i <= (left + right) / 2 && j <= right) {
			if(nums[i] <= nums[j]) {
				numsTmp[key++] = nums[i++];
			} else {
				numsTmp[key++] = nums[j++];
			}
		}
		while(i <= (left + right) / 2) {
			numsTmp[key++] = nums[i++];
		}
		while(j <= right) {
			numsTmp[key++] = nums[j++];
		}
		for(i = left; i <= right; i++) {
			nums[i] = numsTmp[i];
		}
	}
```

## 选择类排序

### 选择排序
```java
/**
 * 选择排序
 * @param nums
 */
public static void chooseSort(int[] nums) {
    if(nums == null || nums.length < 2) {
        return ;
    }
    for(int i = 0; i < nums.length; i++) {
        int min = Integer.MAX_VALUE, index = -1;
        for(int j = i; j<nums.length; j++) {
            if(nums[j] < min) {
                min = nums[j];
                index = j;
            }
        }
        if(index != -1) {
            nums[index] = nums[i];
            nums[i] = min;
        }
    }
}
```


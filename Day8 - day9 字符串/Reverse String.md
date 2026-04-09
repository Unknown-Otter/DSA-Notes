https://leetcode.com/problems/reverse-string/description/
```java []
class Solution {
    public void reverseString(char[] s) {
        int left  =0;
        int right = s.length-1;

        while(left < right){

            char temp = '\0';
            temp  = s[left];
            s[left] = s[right];
            s[right] = temp;

            left++;
            right--;
        }
    }
}
```

https://leetcode.com/problems/reverse-string-ii/submissions/1973909162/?submissionId=1973909162
```java []
class Solution {
    public String reverseStr(String s, int k) {
        char[] ch = s.toCharArray();
        int len = ch.length; 

        for(int i=0; i< len; i += 2*k){
            int left = i; 
            int right = Math.min(i + k - 1, len - 1); 

            while(left < right){
                char temp = ch[left]; 
                ch[left] = ch[right]; 
                ch[right] = temp; 
                left++;
                right--;
            }
        }
        return new String(ch);
    }
}
```

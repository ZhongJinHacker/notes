```java
public class ListNode {
  int num;
  ListNode next;
}

public ListNode getLastNode(ListNode head, unsigned int k) {
  if (head == null) return null;
  ListNode firstNode = head;
  ListNode secondNode = head;
  for (int i = 0; i < k, i++) {
    if (secondNode.next != null) {
      secondNode = second.next;      
    } else {
      return null;
    }
  }
  
  while (secondNode.next != null) {
    firstNode = firstNode.next;
    seondNode = secondNode.next;
  }
  return firstNode;
}
```


# LRU-Cache-with-TTL
 LRU (Least Recently Used) Cache is implemented by the combination of HashMap and LinkedList Data Structure. 
 ## LRU Cache Solution
 - Define ListNode and DoubleLinkedList class. For DoubleLinkedList class methods like tailPop(), move2Head(), headInsert() should be implemented.
 - Define LRUCache class with capacity, HashMap, DoubleLinkedList(head, tail) delared. The map stores the key and the listnode as value. The DoubleLinkedList stores  key and value.
 - put: If the map contains the key, move the node with the key to head and update the map value; else put the new key, value pair into map and if the map size exceeds the capacity, taipop the list and remove the tail pop node key from map.
  - get: If the map doesn't contains the key, simply return -1; if the map contains the key, move the node to head and return the value of the node.
```java
class ListNode {
    int key;
    int val;
    ListNode prev;
    ListNode next;
    ListNode (int key, int val) {
        this.key = key;
        this.val = val;
    }
    
}
class DoubleLinkedList {
    ListNode head;
    ListNode tail;
    DoubleLinkedList(ListNode head,ListNode tail ) {
        this.head = head;
        this.tail = tail;
    }
    public void move2Head( ListNode node) {
        ListNode prev = node.prev;
        ListNode next = node.next;
        prev.next = next;
        next.prev = prev;
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
        
    }
    public void insert(ListNode node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    public ListNode pop() {
        ListNode temp = tail.prev;
        if (temp.prev == null) {
            return null;
        }
        temp.prev.next = tail;
        tail.prev = temp.prev;
        return temp;
    }
}
class LRUCache {
    int cap = 0;
    Map<Integer, ListNode> map;
    ListNode head;
    ListNode tail;
    DoubleLinkedList list;
    
    public LRUCache(int capacity) {
        map = new HashMap<>();
        cap = capacity;
        head = new ListNode(0, 0);
        tail = new ListNode(0, 0);
        head.next = tail;
        tail.prev = head;
        list = new DoubleLinkedList(head, tail);
    }
    
    public int get(int key) {
        if (!map.containsKey(key)) {
            return -1;
        }
        list.move2Head(map.get(key));
        return map.get(key).val;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) {
            list.move2Head(map.get(key));
            map.get(key).val = value;
            
        } else {
            ListNode node = new ListNode(key, value);
            map.put(key, node);
            list.insert(node);
            if(map.size() > cap) {
                ListNode n = list.pop();
                if (n != null) {
                     map.remove(n.key);
                }
               
            }
        }
        
        
    }
}

```

## LRU Cache with TTL Solution
- For each node add an adidtional field "expiry", which is the creation time + ttl.
- On lookup check if the node has expired by comparing current_time with expiry.
- get: If it has expired delete the node and return -1. If its has not expired, then return the value found.
The new ListNodes have a newly added attribute - expiretime:
```java
class ListNode {
    //...
    long expireTime;//the time to expire, to be compared with the current system time
    public ListNode(int key, int value, long expireTime) {
        this.value = value;
        this.key = key;
        this.expireTime = expireTime;
    }
}
//...
class LRUCache {
    //...
    long time = time;
    public LRUCache(int capacity) {
       //...
       head = new ListNode(0, 0,System.currentTimeMillis() + time);
       tail = new ListNode(0, 0, System.currentTimeMillis() + time);
       //...
    }
    
    public int get(int key) {
       //...
        if (map.get(key).expireTime < System.currentTimeMillis()) {
            map.remove(key);
            list.remove(map.get(key));
            return -1;
        }
        list.move2Head(map.get(key));
        map.get(key).expireTime = System.currentTimeMillis() + time;
        return map.get(key).val;
    }
    
    public void put(int key, int value) {
        //...
        }
        
        
    }
}
```
## Comments
 - This is kind of the lazy eviction - it will delete expired key-value pair only when it call the get method. So an improvement is that we can make it eager eviction by implementing another thread removing all expired entries.
 - So here comes the multi-thread scenario where we should use ConcurrentHashMap instead of HashMap in to store the key-value pair.
 - To improve the eviction thread, we can implement a PriorityQueue for the ListNodes to get the expired nodes quickly. eg.
```java
private PriorityQueue<ListNode> expireQueue = new PriorityQueue<>((a,b)->a.epireTime - b.expireTime);
```

## Reference:
https://blog.csdn.net/SDDDLLL/article/details/106113970

#### 问题1：测试hashmap中链表与红黑树互相转换的时机。

```java
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
```

binCount从0开始计数，当链表内有8个节点的后，`p.next = newNode(hash, key, value, null);`又新加了一个节点，即当链表内有9个节点的时候，调用treeifyBin进行树化。

```java
            if (root == null
                || (movable
                    && (root.right == null
                        || (rl = root.left) == null
                        || rl.left == null))) {
                tab[index] = first.untreeify(map);  // too small
                return;
            }
```

当从map中remove元素时，如果满足给定的条件，即根节点为空，或者在movable情况下，根节点的左右节点有一个为空，或者根节点的左节点的左节点为空时，认为红黑树节点数过少，退化为链表。

```java
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }

            if (loHead != null) {
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
```

当table进行扩容时，红黑树会进行拆分，如果此时会使用6作为判断条件，如果单节点数量小于阈值6，则退化为链表。

#### 问题2：测试红黑树退化链表过程，遇到debug和run运行结果不一致的问题。

```java
    @Test
    public void test() throws Exception {

        Map<Per, String> map = new HashMap<>();
        for (int i = 0; i < 45; i++) {
            map.put(new Per(i), "value" + i);
        }

        for (int i = 0; i < 45; i++) {
            map.remove(new Per(i));
            Field table = map.getClass().getDeclaredField("table");
            table.setAccessible(true);
            Object[] objects = (Object[]) table.get(map);
            System.out.println("###################################");
            System.out.println(objects.length + " " + (i + 1));
            for (int j = 0; j < objects.length; j++) {
                if (objects[j] != null) {
                    System.out.println(objects[j].getClass() + " " + j);
                }
            }
            System.out.println("###################################");
        }

    }

    static class Per implements Comparable<Per> {
        Integer id;

        public Per(Integer id) {
            this.id = id;
        }

        @Override
        public int hashCode() {
            return id % 5;
        }

        @Override
        public boolean equals(Object obj) {
            return id - ((Per) obj).id == 0;
        }

        @Override
        public int compareTo(Per o) {
            return this.id - o.id;
        }
    }
```

解决：作为key的类实现Comparable接口，因为红黑树涉及排序。
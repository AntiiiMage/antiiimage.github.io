## First take a look at HashMap
HashMap stores the Node instances in an array and not as key-value pairs
```java
/**
* The table, resized as necessary. Length MUST Always be a power of two.
*/
transient HashMap.Node<K, V>[] table;
```
## What is Node
```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        ......
    }
```
## How Put() works
```java
     /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length; //Initiate table and assign length to n
        if ((p = tab[i = (n - 1) & hash]) == null) //Generates index based on the re-generated hashcode and length of the data structure.
            tab[i] = newNode(hash, key, value, null); //Insert new node if first node not exists at index i
        else { // Frist node p exists
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))//If first node's key equals to target key
                e = p; //Set first node as current node 
            else if (p instanceof TreeNode) // if it's already a tree, add tree node
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value); // Set newly-added tree node as current node
            else { //If the element is linked list(node)
                for (int binCount = 0; ; ++binCount) { // Searching node whose key matched target key
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null); //If not matched, create node at end and set as current node
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);//Convert linked list as  tree if  lenght exceed limit
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break; // if current node's key matched with target, break
                    p = e; //move to next node
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value; // replace value
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

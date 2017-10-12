## First take a look at HashMap
HashMap stores the Node instances in an array and not as key-value pairs
```java
transient HashMap.Node<K, V>[] table; //The table, resized as necessary. Length MUST Always be a power of two.
transient int size; //The number of key-value mappings contained in this map.
/**
* The number of times this HashMap has been structurally modified, is used to make iterators on Collection-views
* of the HashMap fail-fast
*/
transient int modCount; 
int threshold; //The next size value at which to resize (capacity * load factor).
final float loadFactor;
static final float DEFAULT_LOAD_FACTOR = 0.75F;
/**
* The bin count threshold for using a tree rather than list for a
* bin.  Bins are converted to trees when adding an element to a
* bin with at least this many nodes. The value must be greater
* than 2 and should be at least 8 to mesh with assumptions in
* tree removal about conversion back to plain bins upon
* shrinkage.
*/
static final int TREEIFY_THRESHOLD = 8;
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
                        break; // if existing node's key matched with target, break
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
        ++modCount; //increment the number of structural modification
        if (++size > threshold) //Increment map element size and compare with threshold
            resize(); // Resize if execeed threshold
        afterNodeInsertion(evict);
        return null;
    }
```
## How Get() works
```java
    /**
     * Implements Map.get and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {//Find first node at index (n-1) & hash
            if (first.hash == hash && // Always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first; //Return exist node if first node matched
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);//Find existing node in tree, null if no matching
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e; // Find matched existing node
                } while ((e = e.next) != null); // Traverse the linked list
            }
        }
        return null;
    }
```

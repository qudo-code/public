
# 33 Times Hash
This is Daniel J. Bernstein's popular 'times 33' hash function 
```ts
const hash = (text) => {
    let hash = 5381;
    let index = text.length;
    
    while (index) {
      hash = (hash * 33) ^ text.charCodeAt(--index);
    }
  
    return hash >>> 0;
}
```

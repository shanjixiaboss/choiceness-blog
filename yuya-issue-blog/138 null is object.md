 return typeof ret === 'object' ? ret : obj;  
=>
 return typeof ret === 'object' ? (ret || obj) : obj;  
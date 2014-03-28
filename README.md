# Generators are basically iterators -(minus) Iterator class complexity 
(example from: http://pl1.php.net/manual/en/language.generators.comparison.php)

    
```PHP
function getLinesFromFile($fileName) {
    if (!$fileHandle = fopen($fileName, 'r')) {
        return;
    }
 
    while (false !== $line = fgets($fileHandle)) {
        yield $line;
    }
 
    fclose($fileHandle);
}
```


versus

```PHP
    class LineIterator implements Iterator {
      protected $fileHandle;
   
      protected $line;
      protected $i;
   
      public function __construct($fileName) {
          if (!$this->fileHandle = fopen($fileName, 'r')) {
              throw new RuntimeException('Couldn\'t open file "' . $fileName . '"');
          }
      }
   
      public function rewind() {
          fseek($this->fileHandle, 0);
          $this->line = fgets($this->fileHandle);
          $this->i = 0;
      }
   
      public function valid() {
          return false !== $this->line;
      }
   
      public function current() {
          return $this->line;
      }
   
      public function key() {
          return $this->i;
      }
   
      public function next() {
          if (false !== $this->line) {
              $this->line = fgets($this->fileHandle);
              $this->i++;
          }
      }
   
      public function __destruct() {
          fclose($this->fileHandle);
      }
  }
```

## But you can't rewind them!

```PHP
function _range($start, $end) {
  while($start <= $end) {
    yield $start;
    $start++;
  }
}

$mySet = _range(5,7);

foreach($mySet as $value) {
  var_dump($value);
}
foreach($mySet as $value) {
  var_dump($value);
}
```

    int(5)
    int(6)
    int(7)
    PHP Fatal error:  Uncaught exception 'Exception' with message 'Cannot traverse an already closed generator'
    
Which can be good ;)
    
## Clone isn't working

    PHP Fatal error:  Trying to clone an uncloneable object of class Generator
    
Despite of what php.net claims. 


## Also you can not use next(), current() on generators. 


# return? no-no

```PHP
// Can't return value from generator:
return 2;

// but ok to stop generator from processing
return;
```


# Question: 

what this will produce?
```PHP
$mySet = _range('a','z');

foreach($mySet as $value) {
  var_dump($value);
}
```

# Send stuff to your generator.

The only interesting example I've found is batch processing, which can be interesting.


```PHP

function fetchSelect($selectObject) {
  $start = 0;
  while(true) {
     $howMuch = yield;
     $select->offset($start)->limit($howMuch);
     $result = $select->fetch();
     yield $result
     $start += $howMuch +1; // move offset
  }
}

/** we need to do something super-complex in PHP side to find interesting records **/
function batchProcess(Generator $selectGenerator) {
  $results = $selectGenerator->send(1000);
  while( /** still need more **/) {
     $results = $selectGenerator->send(500);
     if($result->isInteresting()) {
       yield $result;
       return;
     }
  }
}

foreach(batchProcess(fetchSelect($someSelectObject)) as $item) {
// ...
}

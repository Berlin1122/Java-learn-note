## 创建流的方式

- 第一种创建流的方式：使用Stream.of()方法

  ```java
  public static void testStreamOf(){    
      Stream<String> stringStream = Stream.of("he","llo"," world");
  }
  ```

- 第二种创建流的方式：调用集合的stream()方法

  ```java
  public static void testStreamMethod(){    
      List<String> strList = new ArrayList<>();    
      strList.add("he");    
      strList.add("llo");    
      strList.add("world");    
      Stream<String> stringStream = strList.stream();
  }
  ```

- 第三种创建流的方式：产生随机数流(IntStream,DoubleStream,LongStream)

  ```java
  public static void testRandomStream(){    
      Random r = new Random();    
      IntStream intStream = r.ints(4,3,10);
  }
  ```

- 第四种创建流的方式：实现Supplier接口,结合Stream.generate()方法生成流对象

  ```java
  //随机产生单词的类
  class Randomwords implements Supplier<String> {    
      private List<String> words = new ArrayList<>();    
      Random rand = new Random(47);    
      public Randomwords(String fname) throws IOException {        
          //将文件中的内容按行读到List中        
          List<String> lines = Files.readAllLines(Paths.get(fname));        
          //跳过第一行        
          for (String line : lines.subList(1,lines.size())) {            
              for (String word:line.split("[ .?,]+")){                
                  //把每行内容切割成单词并加入到words列表                			
                  words.add(word.toLowerCase());            
              }        
          }    
      }    
      @Override    
      public String get() {        
          return words.get(rand.nextInt(words.size()));    
      }
  }
  ```

  ```java
  public static void testSupplierAndGenerate() throws IOException {    
      Stream<String> stringStream = Stream.generate(new Randomwords("test.txt"));
  }
  ```

- 第五种创建流的方式：使用Stream.Builder

  ```java
  class FileToWordsBuilder {    
      Stream.Builder<String> builder = Stream.builder();    
      public FileToWordsBuilder(String fname) throws IOException {        
          //将文件的每一行读到内存中，跳过第一行，对每一行进行单词切割，并把单词加入builder列表   
          Files.lines(Paths.get(fname)).skip(1).forEach(line->{            
              for (String w : line.split("[ .?,]+"))                
                  builder.add(w);        
          });    
      }   
      public Stream<String> stream(){        
          return builder.build();   
      }
  }
  ```

  ```java
  public static void testStreamBuilder() throws IOException {    
      FileToWordsBuilder builder = new FileToWordsBuilder("test.txt");    
      Stream<String> stringStream = builder.stream();
  }
  ```

- 第六种创建流的方式：使用Arrays.stream(T[] arr)方法

  ```java
  public static void testArraysStream(){    
      int[] ints = {1,2,3,4,5,6,7};    
      IntStream integerStream = Arrays.stream(ints);
  }
  ```



## 流的中间操作

```java
//流的中间操作
public class Demo4 {    
    public static void main(String[] args) throws IOException {        
        //testPeek();        
        //testSorted();        
        //testDistinct();        
        //testFilter();        
        //testMap();        
        //testMapToInt();        
        //testFlatMap();    
    }    
    //peek()操作，对元素进行声明函数操作，用于查看不可用于修改    
    public static void testPeek() throws IOException {        
        new FileToWordsBuilder("test.txt").stream()                
            .limit(3).map(i->i+" ")                
            .peek(System.out::print)                
            .map(String::toLowerCase)                
            .peek(System.out::print)                
            .map(String::toUpperCase)                
            .forEach(System.out::print);    
    }   
    
    //sorted()排序操作    
    public static void testSorted(){        
        int[] arr = {2,4,1,3,8,4,3};        
        //1、默认不传比较器        
        Arrays.stream(arr).sorted().forEach(System.out::println);    
    }   
    
    //distinct()去除重复元素    
    public static void testDistinct(){        
        int[] arr = {2,4,1,3,8,4,3};     
        Arrays.stream(arr).distinct().forEach(System.out::println);    
    } 
    
    //filter(func())过滤掉不满足要求的元素    
    public static void testFilter(){        
        int[] arr = {2,4,1,3,8,4,3};        
        Arrays.stream(arr).filter(x-> x > 3).forEach(System.out::println);    
    }   
    //map(func()),对元素进行声明的函数中操作    
    public static void testMap(){        
        int[] arr = {2,4,1,3,8,4,3};        
        Arrays.stream(arr).map(n->n+1).forEach(System.out::println);    
    }  
    //mapToInt(),将元素转换为int类型    
    public static void testMapToInt(){        
        String[] strings = {"23","34","90"};   
        Arrays.stream(strings).mapToInt(Integer::parseInt)
            .forEach(n->System.out.format("%d ",n));    
    }    
    //flap(),流扁平化：对流中的每个元素创建一个满足指定要求的流。    
    public static void testFlatMap(){        
        int[] arr = {1,2,3};       
        Random random = new Random();        
        Arrays.stream(arr).flatMap(i->random.ints(i,5,10))        
            .forEach(n->System.out.format("%d ",n));        
        //最终会产生6个元素    
    }
}
```



## 流的终端操作

```java
public class Demo5 {   
    
    //toArray();将流转化为数组    
    static void testToArray(){        
        int[] rints = new Random(47).ints(0, 1000).limit(100).toArray();    
    }  
    
    //forEach(func())对每个元素执行指定操作    
    static void testForEach(){       
        new Random(47).ints(0, 1000).limit(100).forEach(System.out::print);    
    }  
    
    //由数组转换而来的stream转为list    
    static void testCollect(){        
    	int[] arr = {1,2,3};        
        ArrayList<Integer> list = Arrays.stream(arr)
            .collect(ArrayList<Integer>::new,ArrayList::add,ArrayList::addAll);    
    }    
    
    //由列表转换的stream转为list    
    static void testCollectorsToXXX(){       
        List<String> strings = new ArrayList<>();     
        strings.add("hello");      
        strings.add("world");        
        List<String> strings2 = strings.stream().sorted().collect(Collectors.toList());    }}
```
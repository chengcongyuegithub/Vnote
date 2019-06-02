# channelOption等配置
## part1
```
void init(Channel channel) throws Exception {
        final Map<ChannelOption<?>, Object> options = options0();
```
```
public class ChannelOption<T> extends AbstractConstant<ChannelOption<T>> {
```
channelOption就是一些TCP的配置信息,入下图
```
    public static final ChannelOption<Boolean> SO_BROADCAST = valueOf("SO_BROADCAST");
    public static final ChannelOption<Boolean> SO_KEEPALIVE = valueOf("SO_KEEPALIVE");
    public static final ChannelOption<Integer> SO_SNDBUF = valueOf("SO_SNDBUF");
    public static final ChannelOption<Integer> SO_RCVBUF = valueOf("SO_RCVBUF");
    public static final ChannelOption<Boolean> SO_REUSEADDR = valueOf("SO_REUSEADDR");........
```
其中的泛型就表示值的类型
然后我们看public class ChannelOption<T> extends AbstractConstant<ChannelOption<T>> 它是实现AbstractConstant的,这个AbstractConstant继承了Constant,
这是一个接口,直译为常量
```
/**
 * A singleton which is safe to compare via the {@code ==} operator. Created and managed by {@link ConstantPool}.
 */
public interface Constant<T extends Constant<T>> extends Comparable<T> {

    /**
     * Returns the unique number assigned to this {@link Constant}.
     */
    int id();

    /**
     * Returns the name of this {@link Constant}.
     */
    String name();
}

```
## part2
然后我们进入到ConstantPool中去看,这个就相当于是常量创建的类
```
 private final ConcurrentMap<String, T> constants = PlatformDependent.newConcurrentHashMap();  
 private final AtomicInteger nextId = new AtomicInteger(1);
```
```
private T getOrCreate(String name) {
        T constant = constants.get(name);
        if (constant == null) {
            final T tempConstant = newConstant(nextId(), name);
            constant = constants.putIfAbsent(name, tempConstant);
            if (constant == null) {
                return tempConstant;
            }
        }
        return constant;
    }
```
双重检测,并发检测
## part3
```
 private static final ConstantPool<ChannelOption<Object>> pool = new ConstantPool<ChannelOption<Object>>() {
        @Override
        protected ChannelOption<Object> newConstant(int id, String name) {
            return new ChannelOption<Object>(id, name);
        }
    };
```
## part4
```
//tcp数据
final Map<ChannelOption<?>, Object> options = options0();
        synchronized (options) {
            setChannelOptions(channel, options, logger);
        }
//业务数据
        final Map<AttributeKey<?>, Object> attrs = attrs0();
        synchronized (attrs) {
            for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
                @SuppressWarnings("unchecked")
                AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
                channel.attr(key).set(e.getValue());
            }
        }
```
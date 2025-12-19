# Java 8

### 1. Lambda 表达式 - 函数式编程支持

Lambda 表达式允许将函数作为方法参数，使代码更简洁。

```java
// 传统方式
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});

// Lambda 表达式
Collections.sort(names, (a, b) -> a.compareTo(b));

// 更简洁的写法
Collections.sort(names, String::compareTo);

// 实际业务场景：按用户年龄排序
List<User> users = getUserList();
users.sort((u1, u2) -> u1.getAge() - u2.getAge());
```

### 2. Stream API - 流式数据处理

Stream API 提供了声明式的数据处理方式，支持过滤、映射、聚合等操作。

```java
// 传统方式：筛选年龄大于18的用户并获取姓名
List<String> adultNames = new ArrayList<>();
for (User user : users) {
    if (user.getAge() > 18) {
        adultNames.add(user.getName());
    }
}

// Stream 方式
List<String> adultNames = users.stream()
    .filter(user -> user.getAge() > 18)
    .map(User::getName)
    .collect(Collectors.toList());

// 业务场景：订单金额统计
List<Order> orders = getOrders();
// 计算所有已完成订单的总金额
BigDecimal totalAmount = orders.stream()
    .filter(order -> "COMPLETED".equals(order.getStatus()))
    .map(Order::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// 按用户分组统计订单数量
Map<Long, Long> orderCountByUser = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getUserId,
        Collectors.counting()
    ));

// 获取金额最大的订单
Optional<Order> maxOrder = orders.stream()
    .max(Comparator.comparing(Order::getAmount));
```

### 3. 默认方法 - 接口中可以包含默认实现

接口中可以定义默认方法，便于接口演进而不破坏现有实现。

```java
public interface PaymentService {
    // 抽象方法
    void pay(BigDecimal amount);

    // 默认方法
    default void refund(BigDecimal amount) {
        System.out.println("执行默认退款逻辑: " + amount);
        // 提供通用的退款实现
    }

    // 静态方法
    static boolean validateAmount(BigDecimal amount) {
        return amount != null && amount.compareTo(BigDecimal.ZERO) > 0;
    }
}

// 实现类可以选择性覆盖默认方法
public class AlipayService implements PaymentService {
    @Override
    public void pay(BigDecimal amount) {
        System.out.println("支付宝支付: " + amount);
    }

    // 覆盖默认退款方法
    @Override
    public void refund(BigDecimal amount) {
        System.out.println("支付宝退款: " + amount);
        // 支付宝特定退款逻辑
    }
}
```

### 4. 方法引用 - :: 语法简化Lambda

方法引用提供了更简洁的Lambda表达式书写方式。

```java
// 静态方法引用
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> integers = numbers.stream()
    .map(Integer::parseInt)  // 等同于 s -> Integer.parseInt(s)
    .collect(Collectors.toList());

// 实例方法引用
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(System.out::println);  // 等同于 name -> System.out.println(name)

// 构造方法引用
List<User> users = names.stream()
    .map(User::new)  // 等同于 name -> new User(name)
    .collect(Collectors.toList());

// 业务场景：批量转换 DTO
List<OrderDTO> orderDTOs = orders.stream()
    .map(OrderConverter::toDTO)  // 静态方法引用
    .collect(Collectors.toList());
```

### 5. 新的日期时间API - java.time 包

替代老旧的 Date 和 Calendar，提供不可变、线程安全的日期时间类。

```java
// LocalDate - 日期（年月日）
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1990, 5, 20);
long age = ChronoUnit.YEARS.between(birthday, today);

// LocalTime - 时间（时分秒）
LocalTime now = LocalTime.now();
LocalTime meeting = LocalTime.of(14, 30, 0);

// LocalDateTime - 日期时间
LocalDateTime orderTime = LocalDateTime.now();
LocalDateTime tomorrow = orderTime.plusDays(1);

// 业务场景：计算订单过期时间
LocalDateTime createTime = order.getCreatedAt();
LocalDateTime expireTime = createTime.plusHours(24);
boolean isExpired = LocalDateTime.now().isAfter(expireTime);

// 日期格式化
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String timeStr = orderTime.format(formatter);
LocalDateTime parsed = LocalDateTime.parse("2025-12-19 10:30:00", formatter);

// ZonedDateTime - 带时区的日期时间
ZonedDateTime utcTime = ZonedDateTime.now(ZoneId.of("UTC"));
ZonedDateTime shanghaiTime = utcTime.withZoneSameInstant(ZoneId.of("Asia/Shanghai"));

// 业务场景：计算两个日期之间的工作日
public long calculateWorkDays(LocalDate start, LocalDate end) {
    return start.datesUntil(end.plusDays(1))
        .filter(date -> date.getDayOfWeek() != DayOfWeek.SATURDAY
                     && date.getDayOfWeek() != DayOfWeek.SUNDAY)
        .count();
}
```

### 6. Optional - 避免NullPointerException

Optional 提供了优雅的空值处理方式，避免频繁的 null 检查。

```java
// 传统方式
public String getUserEmail(Long userId) {
    User user = userRepository.findById(userId);
    if (user != null) {
        String email = user.getEmail();
        if (email != null) {
            return email.toLowerCase();
        }
    }
    return "unknown@example.com";
}

// Optional 方式
public String getUserEmail(Long userId) {
    return userRepository.findById(userId)
        .map(User::getEmail)
        .map(String::toLowerCase)
        .orElse("unknown@example.com");
}

// 业务场景：获取用户的默认收货地址
public Optional<Address> getDefaultAddress(Long userId) {
    return addressRepository.findByUserId(userId).stream()
        .filter(Address::isDefault)
        .findFirst();
}

// 使用 Optional
Address address = getDefaultAddress(userId)
    .orElseThrow(() -> new BusinessException("用户没有默认地址"));

// ifPresent 和 ifPresentOrElse
Optional<Order> orderOpt = orderRepository.findById(orderId);
orderOpt.ifPresent(order -> {
    log.info("订单号: {}, 金额: {}", order.getOrderNo(), order.getAmount());
});

orderOpt.ifPresentOrElse(
    order -> processOrder(order),
    () -> log.warn("订单不存在: {}", orderId)
);

// Optional 链式调用
String city = Optional.ofNullable(user)
    .flatMap(User::getAddress)
    .map(Address::getCity)
    .orElse("未知城市");
```

### 7. Nashorn JavaScript引擎

在 Java 中执行 JavaScript 代码（注意：Java 15 中已废弃）。

```java
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

public class NashornExample {
    public static void main(String[] args) throws ScriptException {
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");

        // 执行 JavaScript 代码
        Object result = engine.eval("2 + 3 * 4");
        System.out.println("结果: " + result);  // 输出: 14

        // 业务场景：动态规则引擎
        String rule = "amount > 1000 && userLevel >= 3";
        engine.put("amount", 1500);
        engine.put("userLevel", 5);
        Boolean matched = (Boolean) engine.eval(rule);
        System.out.println("规则匹配: " + matched);  // 输出: true
    }
}
```

### 实际业务综合案例

```java
/**
 * 订单统计服务 - 综合运用 Java 8 特性
 */
@Service
public class OrderStatisticsService {

    @Autowired
    private OrderRepository orderRepository;

    /**
     * 统计指定时间段内的订单数据
     */
    public OrderStatisticsDTO getStatistics(LocalDate startDate, LocalDate endDate) {
        // 查询订单
        List<Order> orders = orderRepository.findByCreateTimeBetween(
            startDate.atStartOfDay(),
            endDate.atTime(23, 59, 59)
        );

        // 使用 Stream API 进行统计
        OrderStatisticsDTO statistics = new OrderStatisticsDTO();

        // 总订单数
        statistics.setTotalCount(orders.size());

        // 总金额
        BigDecimal totalAmount = orders.stream()
            .map(Order::getAmount)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        statistics.setTotalAmount(totalAmount);

        // 平均订单金额
        BigDecimal avgAmount = orders.isEmpty() ? BigDecimal.ZERO :
            totalAmount.divide(BigDecimal.valueOf(orders.size()), 2, RoundingMode.HALF_UP);
        statistics.setAvgAmount(avgAmount);

        // 按状态分组统计
        Map<String, Long> countByStatus = orders.stream()
            .collect(Collectors.groupingBy(
                Order::getStatus,
                Collectors.counting()
            ));
        statistics.setStatusDistribution(countByStatus);

        // 获取金额最大的订单
        Optional<Order> maxOrder = orders.stream()
            .max(Comparator.comparing(Order::getAmount));
        maxOrder.ifPresent(order ->
            statistics.setMaxOrderNo(order.getOrderNo())
        );

        // 每日订单统计
        Map<LocalDate, List<Order>> ordersByDate = orders.stream()
            .collect(Collectors.groupingBy(
                order -> order.getCreatedAt().toLocalDate()
            ));

        Map<LocalDate, BigDecimal> dailyAmount = ordersByDate.entrySet().stream()
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                entry -> entry.getValue().stream()
                    .map(Order::getAmount)
                    .reduce(BigDecimal.ZERO, BigDecimal::add)
            ));
        statistics.setDailyAmount(dailyAmount);

        return statistics;
    }

    /**
     * 获取用户的有效订单
     */
    public List<OrderDTO> getUserValidOrders(Long userId) {
        return orderRepository.findByUserId(userId).stream()
            .filter(order -> "COMPLETED".equals(order.getStatus()))
            .filter(order -> order.getAmount().compareTo(BigDecimal.ZERO) > 0)
            .sorted(Comparator.comparing(Order::getCreatedAt).reversed())
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }

    /**
     * 转换为 DTO
     */
    private OrderDTO convertToDTO(Order order) {
        OrderDTO dto = new OrderDTO();
        dto.setOrderNo(order.getOrderNo());
        dto.setAmount(order.getAmount());
        dto.setStatus(order.getStatus());

        // 使用 Optional 处理可能为空的字段
        Optional.ofNullable(order.getRemark())
            .ifPresent(dto::setRemark);

        // 格式化时间
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        dto.setCreatedTime(order.getCreatedAt().format(formatter));

        return dto;
    }
}
```
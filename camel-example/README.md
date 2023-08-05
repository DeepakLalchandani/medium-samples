
### The [Message Filter](https://camel.apache.org/components/3.21.x/eips/filter-eip.html) from the EIP patterns allows you to filter messages.



```java
@RestController
public class TransactionController {

    private final ProducerTemplate producerTemplate;

    public TransactionController(ProducerTemplate producerTemplate) {
        this.producerTemplate = producerTemplate;
    }

    @PostMapping("/incomingTransactions")
    public ResponseEntity<String> handleIncomingTransactions(@RequestBody Transaction transaction) {
        producerTemplate.sendBody("direct:incomingTransactions", transaction);
        return ResponseEntity.status(HttpStatus.OK).body("Transaction received for processing!");
    }

}

@Component
public class TransactionRoute extends RouteBuilder {

    @Override
    public void configure() {
        from("direct:incomingTransactions")
            .choice()
                .when(simple("${body.sourceLocation} == 'HighRiskCountry'"))
                .to("direct:suspiciousTransactions")
            .otherwise()
                .to("jpa:com.omernaci.camelexample.persistence.entity.Transaction");
    }

}

@Component
public class SuspiciousActivityRoute extends RouteBuilder {

    private final TransactionService transactionService;

    public SuspiciousActivityRoute(TransactionService transactionService) {
        this.transactionService = transactionService;
    }

    @Override
    public void configure() {
        from("direct:suspiciousTransactions")
            .process(exchange -> {
                Transaction transaction = exchange.getIn().getBody(Transaction.class);
                transactionService.logSuspiciousActivity(transaction);
            });
    }

}
```

```logcatfilter
o.a.c.impl.engine.AbstractCamelContext   : Apache Camel 4.0.0-RC2 (camel-1) is starting
o.a.c.impl.engine.AbstractCamelContext   : Routes startup (started:2)
o.a.c.impl.engine.AbstractCamelContext   :     Started route1 (direct://suspiciousTransactions)
o.a.c.impl.engine.AbstractCamelContext   :     Started route2 (direct://incomingTransactions)

these log lines show that the Camel context has started successfully, and it is managing two routes (route1 and route2) with their corresponding endpoints
```


- `producerTemplate.sendBody("direct:incomingTransactions", transaction)`: The producerTemplate is an instance of Camel's ProducerTemplate, which allows us to send messages (transactions) to Camel routes for further processing. In this line, we send the received Transaction object to the Camel route with the endpoint `direct:incomingTransactions`.

- The route starts with the `from` method, which sets the starting endpoint of the route. In this case, the route starts from the `direct:incomingTransactions` endpoint, meaning it expects incoming messages (transactions) from a direct component.

- The route uses a `choice` statement, which is a conditional construct in Apache Camel. It allows you to define different routes based on conditions.

- The `when` statement specifies a condition that checks if the sourceLocation property of the incoming message (transaction) is equal to 'HighRiskCountry'.

- If the condition evaluates to true (i.e., the sourceLocation is 'HighRiskCountry'), the route goes to the `direct:suspiciousTransactions` endpoint.

- If the condition in the `when` statement is false (i.e., the sourceLocation is not 'HighRiskCountry'), the route goes to the otherwise statement.

- In the `otherwise` statement, the route continues to the `to` method, which sends the message (transaction) to the jpa component. The jpa component is used to persist the message (transaction) into the database using the Java Persistence API (JPA). 
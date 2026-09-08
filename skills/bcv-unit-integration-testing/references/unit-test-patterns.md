# Unit test patterns (BACC)

Use this reference to write Mockito-based unit tests following BACC conventions.

## Base setup

```java
package pe.interbank.bcv.bacccompliance.output.adapter;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class CceServiceAdapterTest {

    @Mock
    private CustomerSpecialStatusServicePort port;

    @InjectMocks
    private CceServiceAdapter cceServiceAdapter;

    @Test
    void getCceCustomerSpecialStatus_WithRUC_ShouldReturnResponse() {
        // Given
        when(port.getCustomerSpecialStatus(any(), isNull())).thenReturn(mockResponse);

        // When
        var result = cceServiceAdapter.getCceCustomerSpecialStatus("2", "20123456789");

        // Then
        assertNotNull(result);
        verify(port).getCustomerSpecialStatus(any(), isNull());
    }
}
```

Rules:

- `@ExtendWith(MockitoExtension.class)` on the class.
- `@Mock` for ports/interfaces, `@InjectMocks` for the class under test.
- Same package as the production class.
- Given / When / Then comments.

## Publisher test (ArgumentCaptor + reflection for @Value)

```java
@ExtendWith(MockitoExtension.class)
class PowersValidationPublisherTest {

    @Mock
    private MessagePublisherRegistry registry;

    @Mock
    private MessagePublisher messagePublisher;

    private void setLabel(PowersValidationPublisher publisher, String label) throws ReflectiveOperationException {
        Field f = PowersValidationPublisher.class.getDeclaredField("powersValidationReqPublisherLabel");
        f.setAccessible(true);
        f.set(publisher, label);
    }

    @Test
    void publishesMessageWithCorrectDataSubjectAndSchedule() throws Exception {
        when(registry.getPublisher("powersValidationBcvReqPublisher")).thenReturn(messagePublisher);

        PowersValidationPublisher publisher = new PowersValidationPublisher(registry);
        setLabel(publisher, "powers-queue");

        @SuppressWarnings("unchecked")
        ArgumentCaptor<Message> captor = ArgumentCaptor.forClass(Message.class);

        publisher.publishMessage(dto);

        verify(messagePublisher, times(1)).publish(captor.capture());
        Message captured = captor.getValue();

        assertEquals("powers-queue", getFieldValue(captured, "subject"));
    }

    @Test
    void handlesPublishExceptionWithoutThrowing() throws Exception {
        when(registry.getPublisher("powersValidationBcvReqPublisher")).thenReturn(messagePublisher);
        doThrow(new RuntimeException("boom")).when(messagePublisher).publish(any());

        assertDoesNotThrow(() -> publisher.publishMessage(dto));
        verify(messagePublisher, times(1)).publish(any());
    }
}
```

Notes:

- Publishers store `@Value`-injected labels/queues in private fields; use reflection to inject them
  in tests (there is no setter).
- Use `ArgumentCaptor` to assert the built `Message` (subject, data, scheduledTime).
- Cover the exception path (publisher should log, not propagate).

## FeignConfig / interceptor test

```java
class FeignConfigTest {

    private FeignConfig feignConfig;

    @BeforeEach
    void setUp() {
        feignConfig = new FeignConfig();
    }

    @AfterEach
    void tearDown() {
        RequestContextHolder.resetRequestAttributes();
    }

    @Test
    void testRequestInterceptorCopiesHeaders() {
        MockHttpServletRequest mockRequest = new MockHttpServletRequest();
        mockRequest.addHeader("Authorization", "Bearer token123");
        mockRequest.addHeader("X-Trace-Id", "trace-abc");

        RequestContextHolder.setRequestAttributes(new ServletRequestAttributes(mockRequest));

        RequestTemplate template = new RequestTemplate();
        RequestInterceptor interceptor = feignConfig.requestInterceptor();

        interceptor.apply(template);

        assertEquals("Bearer token123", first(template.headers().get("Authorization")));
    }

    @Test
    void testRequestInterceptorWithoutRequestAttributes() {
        RequestContextHolder.resetRequestAttributes();
        RequestTemplate template = new RequestTemplate();
        RequestInterceptor interceptor = feignConfig.requestInterceptor();

        assertDoesNotThrow(() -> interceptor.apply(template));
        assertTrue(template.headers().isEmpty());
    }
}
```

Notes:

- Use `org.springframework.mock.web.MockHttpServletRequest` and `RequestContextHolder`.
- Always `resetRequestAttributes()` in `@AfterEach`.

## Subscriber test

```java
@ExtendWith(MockitoExtension.class)
class ReportTeradataNaturalPersonSubscriberHandlerTest {

    @Mock
    private ReportProcessInCommandPort reportProcessInCommandPort;

    @InjectMocks
    private ReportTeradataNaturalPersonSubscriberHandler handler;

    @Test
    void onMessage_delegatesToCommandPort() {
        TeradataNaturalPerson payload = new TeradataNaturalPerson();

        handler.onMessage(payload);

        verify(reportProcessInCommandPort).processReportTeradataNaturalPerson(any());
    }
}
```

## Anti-patterns

- Do not `@SpringBootTest` for a single unit test.
- Do not mock the class under test.
- Do not assert implementation trivia (e.g. private fields) when a behavior assertion suffices.
- Do not perform real I/O (network, DB, broker).

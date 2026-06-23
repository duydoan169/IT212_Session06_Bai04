Prompt Học Tập Kỹ Thuật Nâng Cao
Hãy đóng vai một Java Technical Mentor có kinh nghiệm dạy Spring WebFlux
và Reactive Programming cho các lập trình viên ở nhiều cấp độ khác nhau.

Bối cảnh học tập:
Tôi là lập trình viên Java đã có kinh nghiệm Spring Boot nhưng chưa từng dùng WebClient.
Tôi cần tích hợp API vận chuyển bên thứ ba (Giao Hàng Nhanh) bất đồng bộ
để không làm chậm luồng xử lý đơn hàng chính của người dùng.

Đừng viết code ngay. Hãy thực hiện lần lượt 3 phần sau:

PHẦN 1 - Level-based Explanation:
Giải thích cơ chế hoạt động bất đồng bộ và Non-blocking của WebClient ở 2 cấp độ:

Cấp độ 1 (Người mới bắt đầu):
Dùng phép ẩn dụ đặt món ăn tại nhà hàng để giải thích sự khác biệt giữa
Blocking (RestTemplate) và Non-blocking (WebClient).
Giải thích tại sao Non-blocking giúp hệ thống phục vụ nhiều khách hàng hơn
mà không cần thêm nhân viên (thread).

Cấp độ 2 (Senior Developer):
Giải thích bằng thuật ngữ chuyên sâu: Event Loop, Reactive Streams,
Mono/Flux, Backpressure, Project Reactor.
Chỉ rõ WebClient xử lý 10,000 request đồng thời khác RestTemplate
ở tầng JVM và OS như thế nào.

PHẦN 2 - Comparative Analysis:
Lập bảng so sánh chi tiết RestTemplate (Blocking) vs WebClient (Non-blocking)
theo đúng các tiêu chí sau:
- Mô hình xử lý (Thread-per-request vs Event Loop)
- Số thread cần thiết khi xử lý 10,000 request đồng thời
- Mức tiêu thụ RAM ước tính cho 10,000 request đồng thời
- Nguy cơ Thread starvation
- Khả năng tích hợp với hệ thống Reactive
- Độ phức tạp học và triển khai

PHẦN 3 - Practical Implementation:
Dựa trên kiến thức đã giải thích ở 2 phần trên,
sinh class Java DeliveryIntegrationService sử dụng WebClient
để gửi thông tin đơn hàng sang API vận chuyển bên thứ ba qua POST request.

Ràng buộc kỹ thuật bắt buộc:
- Java 17, Spring Boot 3.x, Spring WebFlux
- Cấu hình Connection Timeout: 5 giây, Read Timeout: 10 giây
- Cơ chế tự động Retry: 3 lần với Exponential Backoff khi gặp
  lỗi mạng (ConnectException) hoặc Timeout (TimeoutException)
- Sau 3 lần retry thất bại: ném DeliveryIntegrationException
  và ghi log ERROR kèm orderId
- Dùng Lombok @Slf4j để log từng bước ở cấp độ INFO và ERROR
- Cấu hình WebClient bean riêng với Connection Pool (tối đa 500 kết nối)
- Chú thích từng bước bằng tiếng Việt
- Trình bày trong một khối code markdown duy nhất

Phản Hồi Từ AI
Phần 1 - Giải thích đa cấp độ:
==========================================================
CẤP ĐỘ 1: PHÉP ẨN DỤ NHÀ HÀNG
==========================================================

Hãy tưởng tượng một nhà hàng có 2 cách phục vụ khác nhau:

MÔ HÌNH BLOCKING (RestTemplate - Nhà hàng truyền thống):
- Mỗi khách vào nhà hàng được phân công 1 nhân viên phục vụ riêng.
- Nhân viên đó đứng cạnh bàn khách từ lúc gọi món cho đến khi nhà bếp
  nấu xong và mang ra - tức là nhân viên CHỜ hoàn toàn trong suốt thời gian bếp nấu.
- Nếu nhà hàng có 100 nhân viên mà có 101 khách cùng vào một lúc,
  khách thứ 101 phải đứng ở cửa chờ có nhân viên rảnh.
- Đây chính xác là cách RestTemplate hoạt động: 1 request = 1 thread bị giữ
  từ đầu đến cuối, kể cả lúc chờ response từ server bên ngoài.

MÔ HÌNH NON-BLOCKING (WebClient - Nhà hàng hiện đại):
- Nhà hàng chỉ có 10 nhân viên nhưng phục vụ được 1,000 khách cùng lúc.
- Nhân viên nhận order của khách → đưa phiếu vào bếp → NGAY LẬP TỨC
  quay sang phục vụ khách tiếp theo, không đứng chờ.
- Khi bếp nấu xong, họ gọi tên khách qua loa (Event/Callback),
  nhân viên gần nhất rảnh tay sẽ mang món ra.
- Không có nhân viên nào bị "block" - tất cả đều luôn bận rộn phục vụ.

Kết luận dễ hiểu:
RestTemplate = 1 khách cần 1 nhân viên, nhân viên ngồi chờ cùng khách.
WebClient    = 10 nhân viên phục vụ 1,000 khách nhờ không ai phải ngồi chờ.

==========================================================
CẤP ĐỘ 2: THUẬT NGỮ CHUYÊN SÂU - SENIOR DEVELOPER
==========================================================

EVENT LOOP & PROJECT REACTOR:
WebClient được xây dựng trên Project Reactor và Netty,
sử dụng mô hình Event Loop thay vì Thread-per-request.
Netty duy trì một nhóm nhỏ Event Loop Thread (thường = số CPU core × 2).
Mỗi Event Loop thread xử lý hàng nghìn kết nối đồng thời thông qua
Java NIO Selector - một cơ chế multiplexing I/O của OS.

REACTIVE STREAMS & BACKPRESSURE:
WebClient trả về Mono<T> (0 hoặc 1 phần tử) hoặc Flux<T> (0 đến N phần tử).
Đây là implementation của Reactive Streams Specification (Publisher/Subscriber).
Backpressure là cơ chế Subscriber thông báo cho Publisher tốc độ
nó có thể xử lý, ngăn Publisher gửi data nhanh hơn Subscriber tiêu thụ được.
Ví dụ: Consumer xử lý được 100 record/s, Publisher chỉ được phép emit 100/s,
tránh OutOfMemoryError do buffer tràn.

MONO/FLUX VÀ LAZY EXECUTION:
Mono và Flux là lazy - không thực thi gì cho đến khi có subscriber.
Toàn bộ pipeline (filter, map, retry, timeout) chỉ là khai báo.
Khi .subscribe() hoặc .block() được gọi, pipeline mới thực sự chạy.
Điều này cho phép compose pipeline phức tạp mà không tốn tài nguyên
cho đến khi thực sự cần.

THREAD MODEL SO SÁNH:
RestTemplate: request đến → Thread pool lấy 1 thread → thread gọi HTTP
→ thread BLOCK chờ OS network I/O → response về → thread xử lý → trả về.
Thread bị giữ nguyên trong suốt quá trình chờ network (thường 200-2000ms).

WebClient: request đến → Event Loop thread gửi bytes xuống NIO Channel
→ thread KHÔNG BLOCK, đăng ký Selector callback → thread xử lý request khác
→ khi OS báo data ready qua epoll/kqueue → Event Loop thread đọc response
→ trigger Reactor pipeline tiếp tục xử lý.

Phần 2 - Bảng so sánh:
| Tiêu chí                              | RestTemplate (Blocking)              | WebClient (Non-blocking)               |
|---------------------------------------|--------------------------------------|----------------------------------------|
| Mô hình xử lý                         | Thread-per-request                   | Event Loop + NIO                       |
| Thread cần cho 10,000 req đồng thời   | ~10,000 threads                      | ~số CPU core × 2 (thường 8-32 threads) |
| RAM ước tính cho 10,000 req đồng thời | ~10GB (mỗi thread ~1MB stack)        | ~100-200MB (chủ yếu là buffer dữ liệu) |
| Nguy cơ Thread starvation             | Cao - pool cạn kiệt khi tải đột biến | Không có - Event Loop không bị block   |
| Tích hợp hệ thống Reactive            | Không hỗ trợ                         | Tích hợp native với Project Reactor    |
| Độ phức tạp học và triển khai         | Thấp - quen thuộc, dễ debug          | Cao - cần hiểu Reactive paradigm       |

Phần 3 - Class Java DeliveryIntegrationService:
java// ===================== CUSTOM EXCEPTION =====================

/**
* Exception ném ra khi tích hợp API vận chuyển thất bại sau tất cả các lần retry.
  */
  public class DeliveryIntegrationException extends RuntimeException {
  public DeliveryIntegrationException(String orderId, String reason) {
  super(String.format(
  "Tích hợp vận chuyển thất bại sau 3 lần retry - orderId: %s, lý do: %s",
  orderId, reason));
  }

  public DeliveryIntegrationException(String orderId, String reason, Throwable cause) {
  super(String.format(
  "Tích hợp vận chuyển thất bại sau 3 lần retry - orderId: %s, lý do: %s",
  orderId, reason), cause);
  }
  }


// ===================== CẤU HÌNH WEBCLIENT =====================

import io.netty.channel.ChannelOption;
import io.netty.handler.timeout.ReadTimeoutHandler;
import io.netty.handler.timeout.WriteTimeoutHandler;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.reactive.ReactorClientHttpConnector;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.netty.http.client.HttpClient;
import reactor.netty.resources.ConnectionProvider;
import java.time.Duration;
import java.util.concurrent.TimeUnit;

@Configuration
public class WebClientConfig {

    @Value("${delivery.api.base-url:https://api.giaohangnhanh.vn}")
    private String deliveryApiBaseUrl;

    /**
     * Cấu hình WebClient bean với:
     * - Connection Pool tối đa 500 kết nối để tái sử dụng, tránh tạo kết nối mới liên tục
     * - Connection Timeout 5 giây: thời gian tối đa chờ thiết lập kết nối TCP
     * - Read Timeout 10 giây: thời gian tối đa chờ response sau khi gửi request
     */
    @Bean
    public WebClient deliveryWebClient() {

        // Cấu hình Connection Pool với Reactor Netty
        // maxConnections: số kết nối tối đa trong pool
        // pendingAcquireTimeout: thời gian chờ tối đa để lấy kết nối từ pool
        ConnectionProvider connectionProvider = ConnectionProvider.builder("delivery-pool")
            .maxConnections(500)
            .pendingAcquireTimeout(Duration.ofSeconds(10))
            .maxIdleTime(Duration.ofSeconds(30))
            .build();

        // Cấu hình Netty HttpClient với timeout chi tiết
        HttpClient httpClient = HttpClient.create(connectionProvider)
            // Connection Timeout: thời gian thiết lập kết nối TCP tối đa 5 giây
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
            .doOnConnected(conn -> conn
                // Read Timeout: chờ response tối đa 10 giây
                .addHandlerLast(new ReadTimeoutHandler(10, TimeUnit.SECONDS))
                // Write Timeout: gửi request tối đa 5 giây
                .addHandlerLast(new WriteTimeoutHandler(5, TimeUnit.SECONDS))
            );

        return WebClient.builder()
            .baseUrl(deliveryApiBaseUrl)
            .clientConnector(new ReactorClientHttpConnector(httpClient))
            .defaultHeader("Content-Type", "application/json")
            .defaultHeader("Accept", "application/json")
            .build();
    }
}


// ===================== MODEL REQUEST/RESPONSE =====================

import lombok.Builder;
import lombok.Data;
import java.math.BigDecimal;

@Data
@Builder
public class DeliveryRequest {
private String orderId;
private String customerId;
private String recipientName;
private String recipientPhone;
private String deliveryAddress;
private BigDecimal totalAmount;
private int distanceKm;
}

@Data
public class DeliveryResponse {
private String trackingCode;
private String status;
private String estimatedDeliveryTime;
private String message;
}


// ===================== DELIVERY INTEGRATION SERVICE =====================

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.WebClientRequestException;
import org.springframework.web.reactive.function.client.WebClientResponseException;
import reactor.core.publisher.Mono;
import reactor.util.retry.Retry;
import java.net.ConnectException;
import java.time.Duration;

@Slf4j
@Service
@RequiredArgsConstructor
public class DeliveryIntegrationService {

    private final WebClient deliveryWebClient;

    // Endpoint đăng ký đơn hàng vận chuyển
    private static final String CREATE_SHIPMENT_PATH = "/v2/shipping-order/create";

    // Số lần retry tối đa khi gặp lỗi mạng hoặc timeout
    private static final int MAX_RETRY_ATTEMPTS = 3;

    // Thời gian chờ ban đầu giữa các lần retry (sẽ tăng theo Exponential Backoff)
    private static final Duration INITIAL_RETRY_DELAY = Duration.ofSeconds(1);

    /**
     * Gửi thông tin đơn hàng sang API vận chuyển bên thứ ba bất đồng bộ.
     * Hàm trả về Mono<DeliveryResponse> - caller có thể subscribe hoặc
     * chain tiếp với các bước xử lý khác mà không block luồng hiện tại.
     *
     * Cơ chế retry: tự động thử lại tối đa 3 lần với Exponential Backoff
     * (1s → 2s → 4s) khi gặp lỗi kết nối hoặc timeout.
     * Sau 3 lần thất bại, ném DeliveryIntegrationException.
     *
     * @param request Thông tin đơn hàng cần gửi sang đối tác vận chuyển
     * @return Mono<DeliveryResponse> chứa tracking code từ đối tác
     */
    public Mono<DeliveryResponse> createShipment(DeliveryRequest request) {
        log.info("Bắt đầu tạo đơn vận chuyển - orderId: {}, địa chỉ: {}",
            request.getOrderId(), request.getDeliveryAddress());

        return deliveryWebClient
            .post()
            .uri(CREATE_SHIPMENT_PATH)
            .bodyValue(request)
            .retrieve()

            // Xử lý lỗi HTTP 4xx từ API đối tác (lỗi dữ liệu đầu vào)
            .onStatus(
                status -> status.is4xxClientError(),
                clientResponse -> clientResponse.bodyToMono(String.class)
                    .flatMap(errorBody -> {
                        log.error("API vận chuyển trả về lỗi 4xx - orderId: {}, status: {}, body: {}",
                            request.getOrderId(),
                            clientResponse.statusCode(),
                            errorBody);
                        return Mono.error(new DeliveryIntegrationException(
                            request.getOrderId(),
                            "Lỗi dữ liệu đầu vào: " + errorBody));
                    })
            )

            // Xử lý lỗi HTTP 5xx từ API đối tác (lỗi phía server đối tác)
            .onStatus(
                status -> status.is5xxServerError(),
                clientResponse -> {
                    log.warn("API vận chuyển trả về lỗi 5xx - orderId: {}, status: {}",
                        request.getOrderId(), clientResponse.statusCode());
                    return Mono.error(new WebClientResponseException(
                        "Server vận chuyển lỗi",
                        clientResponse.statusCode().value(),
                        clientResponse.statusCode().toString(),
                        null, null, null));
                }
            )

            .bodyToMono(DeliveryResponse.class)

            // Cấu hình Retry với Exponential Backoff
            // Chỉ retry khi gặp lỗi mạng hoặc timeout, KHÔNG retry lỗi 4xx
            .retryWhen(
                Retry.backoff(MAX_RETRY_ATTEMPTS, INITIAL_RETRY_DELAY)
                    .maxBackoff(Duration.ofSeconds(10))
                    .filter(throwable ->
                        // Retry khi mất kết nối mạng
                        throwable instanceof ConnectException ||
                        // Retry khi WebClient không kết nối được (bao gồm timeout kết nối)
                        throwable instanceof WebClientRequestException ||
                        // Retry khi server đối tác lỗi 5xx
                        (throwable instanceof WebClientResponseException wcre &&
                         wcre.getStatusCode().is5xxServerError())
                    )
                    .doBeforeRetry(retrySignal -> log.warn(
                        "Retry lần {} - orderId: {}, lý do: {}",
                        retrySignal.totalRetries() + 1,
                        request.getOrderId(),
                        retrySignal.failure().getMessage()
                    ))
                    // Sau khi hết số lần retry, bọc lỗi thành DeliveryIntegrationException
                    .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
                        log.error(
                            "Đã retry {} lần nhưng vẫn thất bại - orderId: {}, lỗi cuối: {}",
                            MAX_RETRY_ATTEMPTS,
                            request.getOrderId(),
                            retrySignal.failure().getMessage(),
                            retrySignal.failure()
                        );
                        return new DeliveryIntegrationException(
                            request.getOrderId(),
                            retrySignal.failure().getMessage(),
                            retrySignal.failure()
                        );
                    })
            )

            // Log kết quả thành công sau khi nhận được response
            .doOnSuccess(response -> log.info(
                "Tạo đơn vận chuyển thành công - orderId: {}, trackingCode: {}, " +
                "dự kiến giao: {}",
                request.getOrderId(),
                response.getTrackingCode(),
                response.getEstimatedDeliveryTime()
            ))

            // Log nếu có lỗi không được retry (ví dụ: lỗi 4xx)
            .doOnError(error -> {
                if (!(error instanceof DeliveryIntegrationException)) {
                    log.error(
                        "Lỗi không thể retry khi tạo đơn vận chuyển - orderId: {}, lỗi: {}",
                        request.getOrderId(),
                        error.getMessage()
                    );
                }
            });
    }

    /**
     * Gọi bất đồng bộ và không chờ kết quả (fire-and-forget).
     * Dùng trong trường hợp luồng đặt hàng chính không cần chờ
     * xác nhận từ đối tác vận chuyển ngay lập tức.
     * Lỗi sẽ được bắt và log, không propagate lên caller.
     *
     * @param request Thông tin đơn hàng cần gửi
     */
    public void createShipmentAsync(DeliveryRequest request) {
        createShipment(request)
            .subscribe(
                // Xử lý khi thành công
                response -> log.info(
                    "Fire-and-forget thành công - orderId: {}, trackingCode: {}",
                    request.getOrderId(), response.getTrackingCode()
                ),
                // Xử lý khi thất bại - chỉ log, không throw lên caller
                error -> log.error(
                    "Fire-and-forget thất bại - orderId: {}, lỗi: {}",
                    request.getOrderId(), error.getMessage()
                )
            );
    }
}
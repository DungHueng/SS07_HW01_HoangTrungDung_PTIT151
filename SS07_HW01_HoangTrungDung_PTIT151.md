# BÀI 1: Phân tích & Lựa chọn (Prompt viết hàm/code snippet theo mô tả)
## Phương án lựa chọn: B
### phương án B tối ưu nhất dựa trên cấu trúc 4 cấu phần kỹ thuật
- #### Input (Đầu vào)
  * Đầu vào là một chuỗi XML chứa danh sách giao dịch
  * Mỗi giao dịch sẽ bao gồm: (id, amount, status)
- #### Processing (Xử lý)
  * Sử dụng thư viện XML tích hợp sẵn của Java (DOM hoặc SAX Parser)
  * Kiểm tra điều kiện dữ liệu (amount > 0, id không được null hoặc trống)
  * Nếu dữ liệu không hợp lệ thì ném ra InvalidTransactionException
  * Nếu XML sai định dạng thì phải bắt và xử lý ngoại lệ phân tích cú pháp XML
- #### Output (Đầu ra)
  * Hàm phải trả về List<TransactionDTO>
  * TransactionDTO được định nghĩa dưới dạng Java Record gồm (String id, double amount, String status)
- #### Language
  * Promt quy định: Sử dụng java 17, Dùng thư viện XML có sẵn trong Java
### kỹ thuật Dry-run CoT
- Bước 1: Xây dựng Pseudocode
  * #### AI phải mô tả thuật toán bằng ngôn ngữ tự nhiên trước:
  1. Đọc chuỗi XML.
  2. Phân tích các node transaction.
  3. Lấy giá trị id, amount, status.
  4. Kiểm tra dữ liệu hợp lệ.
  5. Tạo TransactionDTO.
  6. Thêm vào danh sách kết quả.
  7. Trả về danh sách.
- Bước 2: Dry-run với dữ liệu lỗi
  * #### Prompt yêu cầu thử nghiệm với XML bị thiếu thẻ đóng.
- Bước 3: Sinh mã nguồn
  * #### Chỉ sau khi hoàn thành phân tích và kiểm tra bằng Dry-run, AI mới sinh code hoàn chỉnh

## Lý do loại trừ phương án A
- Thiếu mô tả Input
- Thiếu ràng buộc Validation
- Thiếu quy định Exception
- Không có Dry-run

## Lý do loại trừ phương án C
- Mâu thuẫn với yêu cầu nghiệp vụ
- Thêm công nghệ không cần thiết
- Tăng độ phức tạp

## Mã nguồn Java hoàn chỉnh do chính AI sinh ra
    import org.w3c.dom.*;
    import org.xml.sax.InputSource;
    import org.xml.sax.SAXException;
    
    import javax.xml.parsers.DocumentBuilder;
    import javax.xml.parsers.DocumentBuilderFactory;
    import javax.xml.parsers.ParserConfigurationException;
    import java.io.StringReader;
    import java.util.ArrayList;
    import java.util.List;
    
    public class TransactionXmlParser {
    
        // DTO sử dụng Java 17 Record
        public record TransactionDTO(
                String id,
                double amount,
                String status
        ) {}
    
        // Ngoại lệ nghiệp vụ
        public static class InvalidTransactionException extends Exception {
            public InvalidTransactionException(String message) {
                super(message);
            }
    
            public InvalidTransactionException(String message, Throwable cause) {
                super(message, cause);
            }
        }
    
        /**
         * Chuyển đổi chuỗi XML thành danh sách TransactionDTO
         */
        public static List<TransactionDTO> parseTransactions(String xml)
                throws InvalidTransactionException {
    
            List<TransactionDTO> transactions = new ArrayList<>();
    
            try {
                DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    
                // Tăng cường bảo mật parser
                factory.setFeature(
                        "http://apache.org/xml/features/disallow-doctype-decl",
                        true
                );
    
                DocumentBuilder builder = factory.newDocumentBuilder();
    
                Document document = builder.parse(
                        new InputSource(new StringReader(xml))
                );
    
                document.getDocumentElement().normalize();
    
                NodeList nodeList =
                        document.getElementsByTagName("transaction");
    
                for (int i = 0; i < nodeList.getLength(); i++) {
    
                    Element element = (Element) nodeList.item(i);
    
                    String id = getTagValue(element, "id");
                    String amountText = getTagValue(element, "amount");
                    String status = getTagValue(element, "status");
    
                    // Kiểm tra ID
                    if (id == null || id.isBlank()) {
                        throw new InvalidTransactionException(
                                "Transaction id must not be empty."
                        );
                    }
    
                    double amount;
    
                    try {
                        amount = Double.parseDouble(amountText);
                    } catch (NumberFormatException ex) {
                        throw new InvalidTransactionException(
                                "Invalid amount format for transaction: " + id,
                                ex
                        );
                    }
    
                    // Kiểm tra số tiền
                    if (amount <= 0) {
                        throw new InvalidTransactionException(
                                "Amount must be greater than 0. Transaction: " + id
                        );
                    }
    
                    transactions.add(
                            new TransactionDTO(
                                    id,
                                    amount,
                                    status
                            )
                    );
                }
    
                return transactions;
    
            } catch (ParserConfigurationException | SAXException ex) {
    
                throw new InvalidTransactionException(
                        "Invalid XML format.",
                        ex
                );
    
            } catch (Exception ex) {
    
                if (ex instanceof InvalidTransactionException) {
                    throw (InvalidTransactionException) ex;
                }
    
                throw new InvalidTransactionException(
                        "Unexpected error while parsing XML.",
                        ex
                );
            }
        }
    
        /**
         * Lấy nội dung của thẻ XML theo tên thẻ
         */
        private static String getTagValue(
                Element element,
                String tagName
        ) {
            NodeList nodeList = element.getElementsByTagName(tagName);
    
            if (nodeList.getLength() == 0) {
                return null;
            }
    
            return nodeList.item(0).getTextContent().trim();
        }
    
        // Ví dụ sử dụng
        public static void main(String[] args) {
            String xml = """
                    <transactions>
                        <transaction>
                            <id>TX001</id>
                            <amount>1000.5</amount>
                            <status>SUCCESS</status>
                        </transaction>
                        <transaction>
                            <id>TX002</id>
                            <amount>500.0</amount>
                            <status>PENDING</status>
                        </transaction>
                    </transactions>
                    """;
    
            try {
                List<TransactionDTO> transactions =
                        parseTransactions(xml);
    
                transactions.forEach(System.out::println);
    
            } catch (InvalidTransactionException e) {
                System.err.println("Lỗi: " + e.getMessage());
            }
        }
      }

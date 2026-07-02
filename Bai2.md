# Bài 2: Tối Ưu Prompt Sinh Docstring & Giải Thích Logic

---

## I. Phân Tích Lỗ Hổng Của Prompt Thô Sơ Hiện Tại

**Prompt hiện tại của thực tập sinh**: *"Giải thích hàm này làm gì và thêm comment vào code giúp tôi: [Đoạn code]"*

### Các lỗ hổng chi tiết:

- **Thiếu Vai trò (Role)**: Không gán vai trò cho AI, dẫn đến AI trả lời ở mức độ chung chung, không dùng ngôn ngữ chuyên ngành phù hợp với domain E-commerce. AI có thể giải thích thuần túy về mặt kỹ thuật (ví dụ: "biến `b` là một số nguyên dùng để so sánh") mà không liên kết được với nghiệp vụ kinh doanh (ví dụ: "b đại diện cho hạng thành viên khách hàng").

- **Thiếu Ngữ cảnh (Context)**: AI không biết đoạn code này thuộc hệ thống gì, module nào, phục vụ nghiệp vụ gì. Hệ quả:
  - AI chỉ đọc code theo cú pháp, không hiểu ý nghĩa kinh doanh đằng sau.
  - Các comment được sinh ra sẽ mang tính mô tả code (what) chứ không giải thích lý do (why).
  - Ví dụ: AI sẽ viết `// Nếu b == 1 thì giảm 10%` thay vì `// Khách hàng hạng Silver được chiết khấu 10%`.

- **Thiếu Ràng buộc (Constraints)**: 
  - Không yêu cầu AI map các biến vô nghĩa (`a`, `b`, `c`) sang tên nghiệp vụ → AI giữ nguyên tên biến cũ trong comment, vẫn gây khó hiểu.
  - Không chỉ rõ chuẩn comment (Javadoc hay inline comment) → AI có thể trộn lẫn hoặc dùng format không chuẩn.
  - Không cung cấp bảng ánh xạ giá trị → AI phải tự đoán ý nghĩa, dẫn đến hallucination.

- **Thiếu Định dạng (Format)**: Không yêu cầu output cụ thể (chỉ code? hay kèm giải thích? dạng bảng? dạng Javadoc?), khiến AI trả về kết quả lẫn lộn, khó sử dụng.

- **Thiếu Mục tiêu rõ ràng (Task)**: Prompt chỉ nói "giải thích" và "thêm comment" — quá chung chung. Không rõ mức độ chi tiết mong muốn, không rõ đối tượng đọc comment là ai (developer mới, QA, hay chính bản thân?).

### Tóm lại:
Prompt thô sơ vi phạm cả 5 thành phần của Prompt Engineering chuẩn, khiến AI trả về kết quả **đúng về cú pháp nhưng sai về ngữ nghĩa nghiệp vụ** — một dạng hallucination rất khó phát hiện vì code "trông có vẻ đúng" nhưng comment không phản ánh logic kinh doanh thực tế.

---

## II. Prompt Tối Ưu (Áp Dụng Đầy Đủ 5 Thành Phần)

```
Vai trò: Đóng vai một Senior Java Developer có kinh nghiệm 10 năm trong lĩnh vực E-commerce, đang thực hiện nhiệm vụ bảo trì hệ thống (legacy code maintenance).

Mục tiêu: Phân tích đoạn code Java dưới đây — hàm `cD()` tính chiết khấu cho đơn hàng — và thực hiện 2 việc:
  1. Giải thích chi tiết logic nghiệp vụ (business logic) mà hàm này triển khai.
  2. Viết lại đoạn code với đầy đủ Javadoc chuẩn và comment inline cho từng dòng phức tạp.

Ngữ cảnh: Đây là đoạn code legacy của hệ thống TechShop (E-commerce), do developer cũ viết và đã nghỉ việc. Code hoạt động đúng nhưng hoàn toàn không có tài liệu. Đội ngũ hiện tại cần hiểu rõ logic để bảo trì và phát triển thêm tính năng. Hàm `cD()` nằm trong module tính giá đơn hàng, được gọi mỗi khi khách hàng thanh toán.

Ràng buộc:
  - BẮT BUỘC ánh xạ (map) các biến vô nghĩa sang tên nghiệp vụ theo bảng sau:
    • Tham số `a` = Tổng giá trị đơn hàng (totalOrderAmount)
    • Tham số `b` = Hạng thành viên (membershipTier): b=1 là Silver, b=2 là Gold
    • Tham số `c` = Voucher freeship (freeShippingVoucher): c=true là có voucher, c=false là không có
    • Biến `d` = Tỷ lệ chiết khấu (discountRate)
    • Giá trị trả về = Số tiền sau khi chiết khấu (finalAmount)
  - Javadoc phải tuân thủ chuẩn Oracle, bao gồm: mô tả hàm, @param cho từng tham số (giải thích ý nghĩa nghiệp vụ), @return mô tả giá trị trả về.
  - Comment inline phải giải thích WHY (tại sao), không chỉ WHAT (cái gì).
  - Giữ nguyên logic code gốc, KHÔNG refactor hay đổi tên biến trong code. Chỉ thêm Javadoc và comment.

Định dạng: Trả về kết quả gồm 2 phần:
  Phần 1 — Bảng ánh xạ biến (dạng Markdown table).
  Phần 2 — Toàn bộ mã nguồn Java đã có Javadoc và comment inline (trong một code block Java duy nhất).

Đoạn code cần phân tích:

public double cD(double a, int b, boolean c) {
    double d = 0;
    if (b == 1) {
        d = 0.1;
    } else if (b == 2) {
        d = 0.2;
    }
    double result = a - (a * d);
    if (c) {
        result = result - 25000;
    }
    if (result < 0) {
        result = 0;
    }
    return result;
}
```

---

## III. Kết Quả Đầu Ra Từ AI (Sau Khi Chạy Prompt Tối Ưu)

### Phần 1: Bảng Ánh Xạ Biến

| Biến gốc | Tên nghiệp vụ              | Ý nghĩa                                                      |
|-----------|------------------------------|---------------------------------------------------------------|
| `a`       | `totalOrderAmount`           | Tổng giá trị đơn hàng (đơn vị: VNĐ)                         |
| `b`       | `membershipTier`             | Hạng thành viên: 1 = Silver, 2 = Gold                        |
| `c`       | `freeShippingVoucher`        | Voucher freeship: `true` = có voucher, `false` = không có     |
| `d`       | `discountRate`               | Tỷ lệ chiết khấu theo hạng thành viên (0.1 = 10%, 0.2 = 20%)|
| `result`  | `finalAmount`                | Số tiền khách hàng phải thanh toán sau khi áp dụng chiết khấu |

### Phần 2: Mã Nguồn Java Đã Có Javadoc & Comment

```java
/**
 * Tính toán số tiền thanh toán cuối cùng sau khi áp dụng chiết khấu theo hạng thành viên
 * và trừ phí vận chuyển nếu khách hàng có voucher freeship.
 *
 * <p>Quy trình tính toán:</p>
 * <ol>
 *   <li>Xác định tỷ lệ chiết khấu dựa trên hạng thành viên (Silver: 10%, Gold: 20%).</li>
 *   <li>Áp dụng chiết khấu lên tổng giá trị đơn hàng.</li>
 *   <li>Nếu có voucher freeship, trừ thêm 25.000 VNĐ (phí vận chuyển cố định).</li>
 *   <li>Đảm bảo số tiền cuối cùng không âm (tối thiểu là 0).</li>
 * </ol>
 *
 * @param a Tổng giá trị đơn hàng trước chiết khấu (totalOrderAmount), đơn vị VNĐ.
 *          Ví dụ: 500000.0 đại diện cho đơn hàng 500.000 VNĐ.
 * @param b Hạng thành viên của khách hàng (membershipTier):
 *          <ul>
 *            <li>{@code 1} = Silver — được chiết khấu 10%</li>
 *            <li>{@code 2} = Gold — được chiết khấu 20%</li>
 *            <li>Các giá trị khác — không được chiết khấu (0%)</li>
 *          </ul>
 * @param c Trạng thái voucher freeship (freeShippingVoucher):
 *          {@code true} nếu khách hàng có voucher freeship,
 *          {@code false} nếu không có.
 * @return Số tiền thanh toán cuối cùng (finalAmount) sau khi trừ chiết khấu và phí ship.
 *         Giá trị trả về luôn >= 0.
 */
public double cD(double a, int b, boolean c) {
    // Khởi tạo tỷ lệ chiết khấu mặc định = 0% (dành cho khách hàng thường, không có hạng thành viên)
    double d = 0;

    // Xác định tỷ lệ chiết khấu dựa trên hạng thành viên (membershipTier)
    if (b == 1) {
        // Khách hàng hạng Silver: được hưởng chiết khấu 10% trên tổng đơn hàng
        d = 0.1;
    } else if (b == 2) {
        // Khách hàng hạng Gold: được hưởng chiết khấu 20% trên tổng đơn hàng
        // (gấp đôi Silver vì đây là chính sách ưu đãi cho khách hàng trung thành)
        d = 0.2;
    }

    // Tính số tiền sau chiết khấu: Tổng đơn hàng - (Tổng đơn hàng × Tỷ lệ chiết khấu)
    // Ví dụ: đơn 500.000 VNĐ, Gold(20%) → 500.000 - (500.000 × 0.2) = 400.000 VNĐ
    double result = a - (a * d);

    // Nếu khách hàng có voucher freeship, trừ thêm 25.000 VNĐ (phí vận chuyển cố định của TechShop)
    // Lý do: Voucher freeship miễn phí vận chuyển, nên khoản phí ship 25.000 VNĐ được khấu trừ khỏi tổng bill
    if (c) {
        result = result - 25000;
    }

    // Đảm bảo số tiền cuối cùng không bao giờ âm
    // Trường hợp xảy ra: đơn hàng nhỏ + chiết khấu lớn + voucher freeship
    // Ví dụ: đơn 20.000 VNĐ, Gold(20%) → 16.000 - 25.000 = -9.000 → trả về 0
    if (result < 0) {
        result = 0;
    }

    // Trả về số tiền khách hàng cần thanh toán thực tế
    return result;
}
```

---

## IV. Đánh Giá So Sánh Kết Quả

| Tiêu chí                           | Prompt thô sơ (cũ)                         | Prompt tối ưu (mới)                              |
|-------------------------------------|---------------------------------------------|---------------------------------------------------|
| Giải thích nghiệp vụ               | ❌ Mô tả thuần kỹ thuật                    | ✅ Gắn liền với ngữ cảnh E-commerce              |
| Ánh xạ biến                        | ❌ Giữ nguyên tên a, b, c                  | ✅ Map đầy đủ sang tên nghiệp vụ                 |
| Javadoc                            | ❌ Không có hoặc sơ sài                    | ✅ Chuẩn Oracle, đầy đủ @param, @return          |
| Comment inline                     | ❌ Chỉ mô tả WHAT                         | ✅ Giải thích WHY với ví dụ cụ thể               |
| Tính ứng dụng thực tế              | ❌ Không dùng được ngay                    | ✅ Copy-paste trực tiếp vào codebase             |

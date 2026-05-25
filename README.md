## Bài tập môn Phát triển ứng dụng với mã nguồn mở - TEE0421
### Họ và tên: Hoàng Kim Ngọc
### MSSV: K225480106053
### Lớp: K58KTP

# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.
## Link gửi bài: [Tại đây](https://docs.google.com/spreadsheets/d/1zftQMj748nRpS-_br4_jdHZocNVvo848zqxCGcTy4uU)

### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
- pull các images về và chạy chúng (up -d)
- Kiểm tra các service đã running ok (ko bị restart liên tục)
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn **Phát triển ứng dụng với mã nguồn mở**
- Truy cập sub-domain3 để cấu hình n8n:
  + tạo tài khoản admin : nhớ điền đúng email
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
  + Create workflow  (home page => overview => Create workflow)
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys
    + cần tạo project mới, sẽ lấy được API KEY
    + Nhập API Key lên giao diện n8n
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
    + Turn on Output Content as JSON : để kết quả trả về dạng json
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
```
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
    + Ignore SSL Issues (Insecure): TURN ON
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger
   
+ Kết quả cuối cùng cần đặt được:
  + từ điện thoại, chát với telegram bot
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
  + f5 wordpress để thấy bài viết mới đã lên sóng.

+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được
+ Nhận xét thành quả đạt được!!!

## Bài làm

1. Bước 1: Chuẩn bị file docker-compose.yml 
- Tạo một thư mục mới trên Ubuntu (ví dụ: btvn04), vào thư mục đó và tạo file docker-compose.yml

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 130929" src="https://github.com/user-attachments/assets/aa55aa86-2c90-4fd6-ab7a-5cc7223f40fc" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 130036" src="https://github.com/user-attachments/assets/8f5c2d41-e46a-4f0d-8210-5244837d4a01" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 130045" src="https://github.com/user-attachments/assets/0ef2f02c-0f4d-4224-bc85-08a6843c8917" />

2. Bước 2: Chạy dịch vụ và Cấu hình Cloudflare Tunnel
   
<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 131201" src="https://github.com/user-attachments/assets/ee0e1451-2cfb-4ecd-ad82-bd1c1e685692" />

3.Bước 3: Cài đặt WordPress và Kiểm tra Database

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 135723" src="https://github.com/user-attachments/assets/f95a6d3b-2cff-4fbd-85fb-d8d273daa4b7" />

<img width="1917" height="1079" alt="Ảnh chụp màn hình 2026-05-25 135806" src="https://github.com/user-attachments/assets/f7ceaa46-1979-4168-aa0b-ab59804acc35" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 135829" src="https://github.com/user-attachments/assets/b905d1e1-fd6c-442a-81fb-73d60fbecaf0" />

<img width="1915" height="1079" alt="Ảnh chụp màn hình 2026-05-25 135834" src="https://github.com/user-attachments/assets/37a188fa-a92c-44e9-9c12-b1c834025094" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 135847" src="https://github.com/user-attachments/assets/8fe32119-5ac5-4268-8453-ea6e55e1ea86" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 140651" src="https://github.com/user-attachments/assets/99772219-e60d-4872-9d0a-87dca498da8d" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 140713" src="https://github.com/user-attachments/assets/de5ce88f-5f54-4b4e-aeed-12f027310590" />

<img width="1914" height="1064" alt="Ảnh chụp màn hình 2026-05-25 141504" src="https://github.com/user-attachments/assets/443850b6-78fb-4ac9-8ea8-cd11ff988888" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 141611" src="https://github.com/user-attachments/assets/025854dc-ccce-4a6d-9759-863c1abbad1e" />

<img width="1916" height="1070" alt="Ảnh chụp màn hình 2026-05-25 141735" src="https://github.com/user-attachments/assets/d8dcdc4a-02c5-40c9-b650-32d88eba9c4e" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 141758" src="https://github.com/user-attachments/assets/6b69b9f4-0890-4614-9e01-9e87fe7cd8b7" />

<img width="1916" height="1079" alt="Ảnh chụp màn hình 2026-05-25 142611" src="https://github.com/user-attachments/assets/07a80f35-c81c-44d2-9e56-a14f6e3e8b8e" />

<img width="1902" height="1049" alt="Ảnh chụp màn hình 2026-05-25 142943" src="https://github.com/user-attachments/assets/ecaea921-ceee-4c53-8245-5f3bc526dccb" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 142954" src="https://github.com/user-attachments/assets/548b066a-fd1b-491c-9da3-abca1c57d6a5" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150052" src="https://github.com/user-attachments/assets/95a3dac6-46b1-41c9-ba1b-8751194459f5" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150219" src="https://github.com/user-attachments/assets/b4a4386d-ec07-49e2-a54f-22bf65ff4aa8" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150226" src="https://github.com/user-attachments/assets/786d37bb-eec4-436e-bfad-235888c1a859" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150358" src="https://github.com/user-attachments/assets/3cf4901d-e879-4db1-915b-eaf3e333ce0a" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150443" src="https://github.com/user-attachments/assets/3da0451e-cf3c-4935-a2c8-a9be8a1e780f" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150626" src="https://github.com/user-attachments/assets/9c9f893c-a3f4-401b-8405-bf7de9772cee" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150715" src="https://github.com/user-attachments/assets/2beef78a-4971-4ecc-95ff-98aa335c5070" />

<img width="1918" height="1079" alt="Ảnh chụp màn hình 2026-05-25 150802" src="https://github.com/user-attachments/assets/c52137dc-6eb2-49df-85e1-a652a4c85de1" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 151339" src="https://github.com/user-attachments/assets/f2d6b559-47cb-4604-9773-c1245e65a88a" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 151348" src="https://github.com/user-attachments/assets/fa776be7-902b-40ae-aeeb-2630f589c698" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 151402" src="https://github.com/user-attachments/assets/f9f28bea-8689-4e47-b5e7-0fe3e7a56865" />

<img width="1919" height="1079" alt="Ảnh chụp màn hình 2026-05-25 151333" src="https://github.com/user-attachments/assets/1a7e9635-cb42-41bb-b77e-821e19c2b6c8" />

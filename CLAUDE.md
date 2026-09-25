# Công tác Games ON – Hải Phòng

Trang web tĩnh 1 file (`index.html`) để chia sẻ thông tin chuyến công tác Hải Phòng
(Games ON) cho đồng nghiệp: lịch bay, khách sạn/ghép phòng, vé máy bay (tải PDF),
suất ăn. Deploy tại repo GitHub `thamtunhut/congtac` (public) — không cần build step,
`index.html` là file chạy trực tiếp.

## Cấu trúc

- `index.html` — toàn bộ trang: CSS + JS + **dữ liệu hardcode** đều nằm trong 1 file
  này (không fetch JSON ngoài, để mở được cả khi double-click file trực tiếp mà
  không cần server — fetch tới file local sẽ bị CORS chặn).
- `Vé máy bay/` — các file PDF vé điện tử + phiếu xác nhận khách sạn, được
  `index.html` link tới bằng đường dẫn tương đối. **Luôn giữ thư mục này đi cùng
  index.html**, không đổi tên file trong này trừ khi cũng sửa lại `TICKETS[].file`
  trong JS.
- `Hướng dẫn xuất hóa đơn di chuyển/` — nguồn cho tab "Hóa đơn": 1 file PDF + 10
  ảnh chụp màn hình gốc, **tất cả đều được commit** (ảnh dùng làm minh họa từng
  bước ngay trong tab). Thứ tự ảnh khớp với từng bước nằm trong
  `GREENSM_STEPS`/`BE_STEPS` (biến JS cuối file, xem mục dưới) — đã đối chiếu thủ
  công từng ảnh với PDF, không suy đoán theo tên file. File PDF vẫn nằm trong repo
  nhưng **không còn nút tải/link nào trỏ tới nó trên trang** (đã bị yêu cầu bỏ nút
  "Tải hướng dẫn đầy đủ (PDF)" và nút "Xem trên Canva" — đừng thêm lại 2 nút này
  trừ khi được yêu cầu). Bảng pháp nhân (VNG Group/VNGGames) gõ tay thẳng vào HTML
  trong `<section id="panel-invoice">`, không phải data-driven — nếu quy trình app
  đổi hoặc thêm pháp nhân mới, sửa trực tiếp ở đó (đọc lại PDF/ảnh gốc hoặc link
  Canva `https://canva.link/6onofzpi71emv0c` nếu anh Vinh gửi bản cập nhật). Mục
  "Talentnet (TBU)" đã bị bỏ theo yêu cầu — đừng thêm lại trừ khi được yêu cầu.
- `Games ON_ Travel & Accommodation CTS.xlsx` — file Excel nguồn (KHÔNG commit lên
  git, xem `.gitignore`) do anh Vinh (VinhHP2) cập nhật. Có 3 sheet:
  - `Plan công tác` — danh sách người đi, ngày đi/về, check-in/out khách sạn.
    **Cột J (Số đêm) dùng merged cells để đánh dấu ai ở ghép phòng với ai** — các
    dòng bị merge chung 1 ô ở cột J = chung phòng. Đây là cách anh Vinh sẽ tiếp
    tục dùng để báo cập nhật ghép phòng trong tương lai.
  - `Ăn uống CTS` — ma trận đăng ký ăn trưa/tối theo domain, theo ngày (05/10–11/10).
  - `Book chuyến bay` — bảng nhóm bay theo ngày, chỉ mang tính tham khảo/đối chiếu.
  - Khi anh Vinh gửi file Excel mới, đọc lại bằng `openpyxl` (Python), đối chiếu
    với dữ liệu hiện có trong `index.html` rồi mới sửa — đừng suy đoán mà không
    xem file gốc.

## Nơi sửa dữ liệu trong `index.html` (đều nằm trong thẻ `<script>` cuối file)

- `FLIGHTS` — thông tin từng chặng bay (giờ, sân bay) theo số hiệu chuyến bay.
- `TICKETS` — 1 vé điện tử (PNR) có thể gồm nhiều hành khách. Mỗi `passenger` có
  thể thêm `returnStatus`:
  - không có field này → đã có vé khứ hồi, hiển thị bình thường trong `legs`.
  - `"pending"` → hiện "⏳ Vé chiều về đang chờ xuất" (vé sẽ ra sau).
  - `"none"` → hiện "Không book vé về" (quyết định không đặt vé về, không phải
    đang chờ).
  - Khi vé về được xuất: thêm leg mới vào `legs` và **xoá** `returnStatus` của
    người đó (ở cả `TICKETS` và `PEOPLE`).
- `PEOPLE` — 1 dòng = 1 lượt đi (Linh NNK và Thư có 2 dòng vì đi 2 đợt: tiền trạm
  27–28/09 và đợt chính 02–12/10). Field `phase` chỉ dùng nội bộ để lọc dữ liệu ăn
  uống (`"main"` = đợt chính từ 02/10, ăn uống chỉ áp dụng nhóm này) — **không hiển
  thị chữ này ra UI** (đã bị yêu cầu bỏ thuật ngữ "Đoàn tiền trạm"/"Đoàn chính").
- `ROOMS` / `SOLO_ROOMS` — danh sách ghép phòng khách sạn chính (đợt 02–12/10).
  `SOLO_ROOMS` để trống nếu ai cũng đã có người ở ghép (mục "Chưa có người ghép
  cùng" tự ẩn khi rỗng). Số phòng luôn hiển thị **"TBU"** cho tới khi khách sạn xác
  nhận thật — lúc đó thêm field `roomNo` vào từng room và sửa `renderRooms()` để
  hiện số thật thay vì "TBU".
- `MEAL_DAYS` / `MEALS` — ma trận suất ăn, key theo domain (đúng domain trong cột
  `Domain` của Excel, kể cả domain có dấu gạch ngang như `"V-ngangt"`).
- `GREENSM_STEPS` / `BE_STEPS` — nội dung tab "Hóa đơn": mỗi bước có `img` (tên
  file ảnh trong `Hướng dẫn xuất hóa đơn di chuyển/`), `txt` (caption), `sub`
  (optional, ghi chú nhỏ). Lưu ý: ảnh chụp màn hình gốc có hiện email cá nhân
  (`hpvinh96@gmail.com`) và 4 số cuối thẻ Visa của anh Vinh (dùng để demo) — nếu
  anh Vinh yêu cầu che/thay ảnh khác thì cần chỉnh sửa ảnh gốc rồi thay file.

## Quy ước đã thống nhất với anh Vinh (đừng đổi lại nếu không được yêu cầu)

- Không dùng chữ "Đoàn tiền trạm" / "Đoàn chính" ở bất kỳ đâu hiển thị cho người
  dùng (kể cả badge, tên nhóm trong vé). Phân biệt 2 đợt chỉ qua ngày tháng.
- Tên đầy đủ: **"Trần Huỳnh Anh Thư"** (không phải "Thu").
- Địa chỉ khách sạn Wink Hai Phong Centre luôn là link Google Maps:
  `https://maps.app.goo.gl/MpEwnu23TJvB7J9q7`.
- Số phòng chưa có → luôn ghi "TBU", không để trống hay ghi "chưa có".
- Không tự suy đoán ai ở ghép phòng với ai nếu không có tín hiệu rõ trong dữ liệu
  (trước đây từng để 3 người "chưa ghép" vì cột J không merge — sau đó anh Vinh
  merge lại thì mới cập nhật).

## Việc còn đang chờ cập nhật (anh Vinh sẽ báo tiếp)

- Số phòng thật của khách sạn Wink Hai Phong Centre (đợt chính 02–12/10) — hiện
  đang "TBU".
- Vé máy bay chiều về (12/10) cho một số thành viên hiện đang `returnStatus:
  "pending"` hoặc `"none"` trong `TICKETS`/`PEOPLE` — cập nhật theo mục hướng dẫn
  ở trên khi có vé mới hoặc có PDF vé mới trong thư mục `Vé máy bay/`.

## Deploy

Repo GitHub: `https://github.com/thamtunhut/congtac` (public, branch `main`), đã
gắn domain riêng `congtac.vinhhp2.site` qua file `CNAME` (do anh Vinh tự thêm qua
GitHub web, không phải mình tạo). Push trực tiếp `index.html` + thư mục
`Vé máy bay/` là đủ để trang chạy được qua GitHub Pages.

**QUAN TRỌNG: KHÔNG tự ý `git commit`/`git push` lên GitHub khi chưa được anh Vinh
yêu cầu rõ ràng trong lượt chat đó.** Cứ sửa file và để đó (commit cục bộ nếu cần
theo dõi, nhưng đừng push) — chỉ push khi anh Vinh nói thẳng kiểu "push lên đi",
"đẩy lên github", v.v. Yêu cầu push một lần không có nghĩa được phép tự push cho
các thay đổi ở lượt chat sau.

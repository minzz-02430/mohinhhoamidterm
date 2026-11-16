# HÒA MA TOÁN · Dashboard

Giao diện React + Vite cho hệ thống quản lý trung tâm Hòa Ma Toán.

## Chạy dự án

```bash
npm install
npm run dev
```

## Kết nối API backend khi phát triển

Vite đọc biến môi trường `VITE_API_BASE_URL` từ `.env.development`. File này hiện đặt URL gốc là `http://localhost:3000/api/v1`; nếu backend chạy ở địa chỉ khác, hãy đổi giá trị trong file rồi khởi động lại Vite.

Hàm `getMyChildrenReports` trong `src/api/reports.js` gọi đúng `GET /reports/my-children`, gửi JWT bằng header `Authorization: Bearer ...`, và trả về `data.reports`. Ví dụ dùng trong component React:

```jsx
import { useEffect, useState } from 'react'
import { getMyChildrenReports } from './api/reports'

function ChildReports({ token }) {
  const [reports, setReports] = useState([])
  const [error, setError] = useState('')

  useEffect(() => {
    const controller = new AbortController()

    async function loadReports() {
      try {
        setError('')
        const result = await getMyChildrenReports(token, controller.signal)
        setReports(result)
      } catch (requestError) {
        if (requestError.name !== 'AbortError') {
          setError(requestError.message)
        }
      }
    }

    loadReports()
    return () => controller.abort()
  }, [token])

  if (error) return <p role="alert">{error}</p>
  return (
    <ul>
      {reports.map(report => (
        <li key={report.report_id}>
          {report.student_name} · {report.lesson_date} · {report.attendance_status}
          {report.tutor_comment ? ` · ${report.tutor_comment}` : ''}
        </li>
      ))}
    </ul>
  )
}
```

`App.jsx` đọc JWT từ `localStorage` với khóa `token`. Khi luồng đăng nhập nhận được JWT trong `data.token`, cần lưu token vào đúng khóa này để dashboard dùng được:

```js
localStorage.setItem('token', loginResponse.data.token)
```

Không đặt JWT thật trong biến `VITE_*` hoặc commit token vào mã nguồn; biến môi trường Vite được đóng gói và hiển thị ở phía trình duyệt. Backend cần đang chạy, có MySQL và tài khoản phụ huynh hợp lệ. `test.html` là demo HTML độc lập, không sử dụng cấu hình `.env.development` của Vite; cần tích hợp lời gọi API riêng nếu muốn nối trang đó.

## Kiểm tra

```bash
npm run build
npm run lint
```

## Các bước đã triển khai

### Bước 1–2 · Layout chung và Quản lý cơ sở

- Layout nền slate sáng với topbar, sidebar riêng cho Quản lý cơ sở và nội dung responsive.
- Dashboard vận hành ngày có 4 KPI, cảnh báo công nợ/trùng lịch/đề xuất nghỉ, thao tác nhanh và nhật ký hoạt động.
- Biểu mẫu học viên, lịch học bù và phiếu thu có tương tác phía client.

### Bước 3 · Giáo viên

- Chuyển vai trò demo trong menu tài khoản để xem sidebar Giáo viên tách biệt.
- Thông tin hồ sơ giáo viên: họ tên, mã GV, giới tính, ngày sinh và cơ sở.
- Thời khóa biểu tuần Thứ 2–Chủ nhật, chia theo 4 ca học; lớp được phân biệt bằng màu cam, xanh dương và vàng.
- Danh sách lớp phụ trách hiển thị sĩ số, phòng học mặc định và lịch học; ô tìm kiếm lọc lớp.
- Có biểu mẫu mô phỏng đề xuất dạy bù/đổi lịch.

### Bước 4 · Phụ huynh

- Không gian phụ huynh dùng topbar điều hướng riêng, không hiển thị sidebar.
- Danh sách nhiều học sinh với khóa học, cơ sở, buổi học sắp tới và tỷ lệ chuyên cần.
- Mở từng hồ sơ để xem điểm danh, bài kiểm tra gần nhất và học phí của riêng học viên đó.
- Có màn hình lịch sử đóng phí và thông tin liên hệ trung tâm.

### Bước 5 · Dữ liệu mẫu

- Dữ liệu giao diện sử dụng tiếng Việt có dấu và ngữ cảnh trung tâm Toán: Toán nâng cao 5A/9A/9B, Toán tư duy 8A và Luyện thi TSA.
- Có hồ sơ giáo viên, lịch học theo ca/phòng, cảnh báo công nợ và xung đột phòng, nhật ký vận hành, kết quả học tập và thông tin cơ sở.
- Lịch sử học phí của gia đình có các khoản đã thanh toán và khoản tháng 10/2026 chờ thanh toán; mức học phí khớp với hồ sơ từng học viên.

## Dữ liệu và giới hạn

Toàn bộ số liệu, hồ sơ, lịch học và giao dịch là dữ liệu mẫu tĩnh dùng cho bản demo. Biểu mẫu chỉ mô phỏng phía client, chưa lưu dữ liệu thật hoặc kết nối API.

Icon giao diện dùng `lucide-react`; Tailwind CSS được nạp trong `src/index.css`.

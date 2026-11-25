# Tích hợp Makeup Lab vào Ray Photo

Makeup Lab có thể chạy riêng hoặc làm màn hình **Makeup** của Ray Photo. Ảnh chỉ được xử lý trong trình duyệt, không gửi lên máy chủ.

## 1. Mở Lab trong Ray Photo

Mở Lab trong một khung (iframe) phủ toàn màn hình:

```html
<iframe id="makeupLab"
        src="https://linhhoang3105-boop.github.io/nang-cap-make-up-Ray/?embed=1"
        style="position:fixed;inset:0;width:100%;height:100%;border:0;z-index:1000"></iframe>
```

Ở chế độ nhúng:
- Lab ẩn nút **Chọn ảnh** và **Lưu ảnh**.
- Thanh trên hiện nút **‹** (quay lại) và **✓ Xong**.

## 2. Gửi ảnh từ Ray Photo sang Lab

Lab báo sẵn sàng bằng tin nhắn `ray-makeup:ready`. Khi nhận được, gửi ảnh sang:

```js
const lab = document.getElementById('makeupLab');
window.addEventListener('message', e => {
  if (e.source !== lab.contentWindow) return;
  const d = e.data || {};
  if (d.type === 'ray-makeup:ready') {
    lab.contentWindow.postMessage({ type: 'ray-makeup:open', image: anhHienTai, name: 'anh.jpg' }, '*');
  }
  if (d.type === 'ray-makeup:done') {
    // d.blob: ảnh PNG đã trang điểm, đúng kích thước gốc (d.width × d.height)
    nhanAnhTuMakeup(d.blob);
    lab.remove();
  }
  if (d.type === 'ray-makeup:cancel') lab.remove();
});
```

- `image` có thể là `Blob`/`File`, chuỗi `data:` URL, `ImageBitmap`, `<canvas>` hoặc `<img>`.
- Lab tự nhận diện khuôn mặt ngay khi nhận ảnh. Gửi kèm `autoDetect:false` nếu không muốn bước này.

## 3. Tin nhắn Lab gửi về

| Tin nhắn | Khi nào | Dữ liệu |
|---|---|---|
| `ray-makeup:ready` | Lab đã tải xong | `version` |
| `ray-makeup:done` | Người dùng bấm **✓ Xong** | `blob` (PNG), `name`, `width`, `height` |
| `ray-makeup:cancel` | Người dùng bấm **‹** | (không có) |

## 4. Gọi trực tiếp (khi hai trang cùng tên miền)

Ray Photo và Lab đều nằm trên `linhhoang3105-boop.github.io` nên có thể gọi trực tiếp:

```js
const L = lab.contentWindow.RayMakeup;
await L.open(blobAnh, 'anh.jpg');       // mở ảnh
await L.detect();                        // nhận diện khuôn mặt
const png = await L.exportBlob();        // lấy ảnh kết quả (mặc định PNG)
L.hasImage(); L.faceCount(); L.version();
```

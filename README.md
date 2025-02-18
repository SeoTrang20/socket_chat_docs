### Kết Nối Tới Socket Server và Hướng Dẫn Cấu Hình Chat Client

#### 1. **Cài Đặt Các Thư Viện**
- **Socket.io**: Thư viện giúp kết nối và giao tiếp thời gian thực với server. 
  - Trang chủ: [https://socket.io](https://socket.io)
- **CryptoJS**: Thư viện hỗ trợ mã hóa và giải mã dữ liệu.
  - Cài đặt: [CryptoJS trên NPM](https://www.npmjs.com/package/crypto-js)

#### 2. **Yêu Cầu Hệ Thống**
- Để các user có thể chat với nhau:
  - Hệ thống cần tạo phòng chat riêng cho từng nhóm người dùng.
  - Khi một user gửi tin nhắn, cần gọi API để lưu trữ tin nhắn vào server.

---

### **Cấu Hình Chat Client**

#### **Bước 1: Mã Hóa User ID**
Mỗi user cần được nhận diện duy nhất bằng `userId`. Để đảm bảo tính bảo mật:
- Mã hóa `userId` bằng khóa bí mật sử dụng **CryptoJS**.

```javascript
import CryptoJS from "crypto-js";

const secretKey = ""; // Khóa bí mật dùng để mã hóa
const encryptedUserId = CryptoJS.AES.encrypt(userId, secretKey).toString();
```

> **Lưu ý:** Khi gửi yêu cầu API tới server, cần thêm `Authorization` header theo định dạng:
> ```bash
> Authorization: Bearer {encryptedUserId}
> ```

---

#### **Bước 2: Kết Nối Tới Socket Server**
Kết nối tới server socket với URL được cung cấp:

- **Server URL**: `wss://****`
- **Cách kết nối**:
```javascript
import { io } from "socket.io-client";

const socket = io(serverUrl, {
  query: { userId: encryptedUserId }, // Gửi userId đã mã hóa trong query
});
```

---

#### **Bước 3: Lắng Nghe Sự Kiện Nhận Tin Nhắn**
Sử dụng socket client để lắng nghe các sự kiện:
```javascript
socket.on("receive", (data) => {
  console.log("Tin nhắn nhận được:", data);
});
```

---

### **API Chat**

#### **Phòng Chat (Conversations)**

1. **Tạo phòng chat**
   - **[POST]** `/api/v1/conversation`
   - **Body yêu cầu**:
   ```json
   {
       "name": null,
       "user_ids": [2] // Danh sách ID người dùng (ngoại trừ ID người tạo)
   }
   ```

2. **Lấy tất cả các phòng chat của người dùng**
   - **[GET]** `/api/v1/conversation/get-all-by-user`

3. **Lấy phòng chat giữa hai người dùng**
   - **[GET]** `/api/v1/conversation/get-one-on-one?userId={userId}`

4. **Lấy danh sách người dùng trong một phòng chat**
   - **[GET]** `/api/v1/conversation/get-all-users/{conversationId}`

5. **Xóa phòng chat từ một phía**
   - **[DELETE]** `/api/v1/conversation/{conversationId}`

5. **Lấy phòng chat bằng id**
   - **[GET]** `/api/v1/get-by-id/{conversationId}`

---

#### **Tin Nhắn (Messages)**

1. **Tạo tin nhắn mới**
   - **[POST]** `/api/v1/message/`
   - **Body yêu cầu**:
   ```json
   {
       "conversation_id": 1,
       "message": ["Hello, world"],
       "type": "text" // hoặc "image"
   }
   ```

2. **Xóa tin nhắn**
   - **[DELETE]** `/api/v1/message/{messageId}`

3. **Lấy tin nhắn trong phòng chat**
   - **[GET]** `/api/v1/message/{conversationId}?page=1&limit=20`


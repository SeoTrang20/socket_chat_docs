### Kết Nối Tới Socket Server và Hướng Dẫn Cấu Hình Chat Client

#### 1. **Cài Đặt Các Thư Viện**

* **Socket.io**: Thư viện giúp kết nối và giao tiếp thời gian thực với server.

  * Trang chủ: [https://socket.io](https://socket.io)
* **CryptoJS**: Thư viện hỗ trợ mã hóa và giải mã dữ liệu.

  * Cài đặt: [CryptoJS trên NPM](https://www.npmjs.com/package/crypto-js)

#### 2. **Yêu Cầu Hệ Thống**

* Để các user có thể chat với nhau:

  * Hệ thống cần tạo phòng chat riêng cho từng nhóm người dùng.
  * Khi một user gửi tin nhắn, cần gọi API để lưu trữ tin nhắn vào server.

---

### **Cấu Hình Chat Client**

#### **Bước 1: Mã Hóa User ID**

Mỗi user cần được nhận diện duy nhất bằng `userId`. Để đảm bảo tính bảo mật:

* Mã hóa `userId` bằng khóa bí mật sử dụng **CryptoJS**.

```javascript
import CryptoJS from "crypto-js";

const secretKey = ""; // Khóa bí mật dùng để mã hóa
const encryptedUserId = CryptoJS.AES.encrypt(userId, secretKey).toString();
```

> **Lưu ý:** Khi gửi yêu cầu API tới server, cần thêm `Authorization` header theo định dạng:
>
> ```bash
> Authorization: Bearer {encryptedUserId}
> ```

---

#### **Bước 2: Kết Nối Tới Socket Server**

Kết nối tới server socket với URL được cung cấp:

* **Server URL**: `wss://****`
* **Cách kết nối**:

```javascript
import { io } from "socket.io-client";

// (đối với người dùng và đại lý)
const socket = io(serverUrl, {
  query: { userId: encryptedUserId },
});

// (đối với hunonic sale)
const socket = io(serverUrl, {
  query: { userId: encryptedUserId, isSale: 1 },
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

   * **\[POST]** `/api/v1/conversation`
   * **Body yêu cầu**:

   ```json
   {
       "name": null,
       "user_ids": [2]
   }
   ```

2. **Lấy tất cả các phòng chat của người dùng**

   * **\[GET]** `/api/v1/conversation/get-all-by-user`

3. **Lấy tất cả các phòng chat của sale**

   * **\[GET]** `/api/v1/conversation/get-all-from-sale`

4. **Lấy phòng chat giữa hai người dùng**

   * **\[GET]** `/api/v1/conversation/get-one-on-one?userId={userId}`

5. **Lấy danh sách người dùng trong một phòng chat**

   * **\[GET]** `/api/v1/conversation/get-all-users/{conversationId}`

6. **Xóa phòng chat từ một phía**

   * **\[DELETE]** `/api/v1/conversation/{conversationId}`

7. **Lấy phòng chat bằng id**

   * **\[GET]** `/api/v1/get-by-id/{conversationId}`

---

#### **Tin Nhắn (Messages)**

1. **Tạo tin nhắn mới (chat giữa các người dùng và đại lý)**

   * **\[POST]** `/api/v1/message/`
   * **Body yêu cầu**:

   ```json
   {
       "conversation_id": 1,
       "message": ["Hello, world"],
       "type": "text"
   }
   ```

2. **Tạo tin nhắn mới (chat với hunonic)**

   * **\[POST]** `/api/v1/message/with-hunonic`
   * **Body yêu cầu**:

   ```json
   {
       "conversation_id": 1,
       "message": ["Hello, world"],
       "type": "text"
   }
   ```

3. **Xóa tin nhắn**

   * **\[DELETE]** `/api/v1/message/{messageId}`

4. **Lấy tin nhắn trong phòng chat**

   * **\[GET]** `/api/v1/message/{conversationId}?page=1&limit=20`

---

### **Đánh Dấu Đã Xem (Seen Messages)**

#### **1. Gửi sự kiện đã xem từ client**

* Khi user đọc xong tin nhắn cuối cùng trong phòng (chat với user và đại lý):

```javascript
await fetch(`/api/v1/seen/${conversationId}?message_id=${lastMessageId}`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${encryptedUserId}`,
    "Content-Type": "application/json",
  },
});
```

* Đối với hunonic (chat với sale hunonic):

```javascript
await fetch(`/api/v1/seen/with-hunonic/${conversationId}?message_id=${lastMessageId}`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${encryptedUserId}`,
    "Content-Type": "application/json",
  },
});
```

#### **2. Sự kiện Socket: `seen`**

* Khi user đọc tin nhắn, server sẽ gửi socket event `"seen"` đến các thành viên còn lại trong phòng.
* **Payload:**

```json
{
  "id": "uuid-v4",
  "conversation_id": 123,
  "user_id": 456,          // Nếu là hunonic hệ thống thì là -1
  "created_at": "2025-05-07T08:13:15.000Z",
  "updated_at": "2025-05-07T08:13:15.000Z"
}
```

* **Client lắng nghe:**

```javascript
socket.on("seen", (data) => {
  console.log("Đã xem:", data);
});
```

---

### **luồng hoạt động**

1. Mã hóa `userId`, gửi lên socket và kèm theo trong `Authorization` header.
2. Lắng nghe socket sự kiện: `receive`, `seen`
3. Khi gửi hoặc xem tin nhắn, gọi API tương ứng:

   * Gửi tin nhắn: `/message`, `/message/with-hunonic`
   * Đánh dấu đã xem: `/seen/:conversation_id`, `/seen/with-hunonic/:conversation_id`
4. Server tự xử lý broadcast socket cho toàn bộ thành viên phòng.

---

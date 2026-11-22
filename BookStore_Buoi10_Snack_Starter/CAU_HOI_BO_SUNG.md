# Trả lời câu hỏi bổ sung · Buổi 10

### Câu 1: Vì sao khi đổi numColumns của FlatList phải đổi cả key? Bỏ key thì chuyện gì xảy ra?
**Trả lời:** FlatList cố định bố cục hàng theo số cột ngay khi khởi tạo và không hỗ trợ thay đổi `numColumns` động trong cùng một instance. Việc đổi `key` (ví dụ `key={String(numColumns)}`) buộc React unmount FlatList cũ và mount lại FlatList mới với số cột mới. Nếu bỏ `key`, khi đổi số cột app sẽ báo lỗi đỏ: *"Changing numColumns on the fly is not supported"*.

---

### Câu 2: Dữ liệu nào nên để trong useState của màn hình, dữ liệu nào nên đưa vào Redux? Lấy ví dụ từ BookStore.
**Trả lời:**
- **useState:** Dùng cho dữ liệu cục bộ (UI state), chỉ một màn hình sử dụng và có thể mất đi khi rời màn. Ví dụ: từ khóa tìm kiếm (`keyword`), chuyển layout (`isTile`), danh mục đang chọn (`category`).
- **Redux:** Dùng cho dữ liệu toàn cục (global state) mà nhiều màn hình cùng đọc/ghi và cần đồng bộ liên tục. Ví dụ: `cart` (màn Chi tiết thêm, màn Giỏ hàng tăng/giảm/xoá, badge trên tab Sách/Giỏ hàng hiển thị số lượng) và `favorites` (màn Chi tiết bật/tắt yêu thích, màn Yêu thích hiển thị danh sách).

---

### Câu 3: Trong createSlice được viết `state.items.push(...)` và `found.quantity += 1`. Vì sao viết như vậy được, trong khi ở component bình thường không được sửa trực tiếp state?
**Trả lời:** Redux Toolkit tích hợp sẵn thư viện **Immer**. Khi thực thi reducer, Immer cung cấp một bản nháp (`draft state`) và tự động ghi nhận các thao tác biến đổi trực tiếp này để tạo ra một state mới hoàn toàn bất biến (immutable). Ngược lại, trong component React bình thường không có Immer; việc mutate trực tiếp `state` sẽ làm cùng tham chiếu cũ, React không nhận ra thay đổi để re-render giao diện.

---

### Câu 4: `selectIsFavorite(id)` trả về một hàm. Vì sao viết như vậy? Danh sách yêu thích chỉ lưu id mà không lưu cả đối tượng sách thì có lợi gì?
**Trả lời:**
- `selectIsFavorite(id)` là một **curried selector** (hàm trả về hàm nhận `RootState`), giúp truyền được tham số `id` vào đồng thời vẫn tương thích với kiểu selector của hook `useAppSelector`.
- Việc chỉ lưu danh sách `id` thay vì lưu toàn bộ đối tượng `Book` tuân thủ nguyên tắc chuẩn hóa dữ liệu (**Single Source of Truth**), tiết kiệm bộ nhớ và tránh lệch dữ liệu: khi giá hay thông tin sách thay đổi thì màn Yêu thích vẫn luôn hiển thị thông tin mới nhất từ nguồn dữ liệu gốc.

---

### Câu 5: BookCard dùng React.memo, còn onSelect ở Home được bọc bằng useCallback. Nếu bỏ useCallback thì React.memo còn tác dụng không? Giải thích.
**Trả lời:** Nếu bỏ `useCallback`, mỗi khi HomeScreen re-render (ví dụ khi gõ phím tìm kiếm hay chuyển category), hàm `onSelect` sẽ được tạo mới với một địa chỉ tham chiếu mới. Do `React.memo` so sánh nông (`shallow comparison`) các props, nó sẽ thấy prop `onSelect` bị thay đổi và buộc phải re-render lại toàn bộ các `BookCard`, làm mất tác dụng tối ưu hiệu năng của `React.memo`.

---

### Câu 6 (Bonus): Một createAsyncThunk có ba hành động pending, fulfilled, rejected. Mỗi hành động làm thay đổi những trường nào của state trong booksSlice?
**Trả lời:**
- `fetchBooks.pending`: Đặt `status = "loading"` và xóa lỗi cũ `error = null`.
- `fetchBooks.fulfilled`: Đặt `status = "succeeded"` và lưu danh sách sách tải về vào `items = action.payload`.
- `fetchBooks.rejected`: Đặt `status = "failed"` và gán thông báo lỗi vào `error = action.payload ?? "Không tải được dữ liệu"`.

# Position:

- **Relative** : Lấy chính nó làm gốc toạ độ ta có thể dùng các thuộc tính top, bottom, left, right để căn chỉnh
- **Absolute** : Lấy thằng cha gần nhất có thuộc tính position để làm gốc toạ độ ta có thể dùng các thuộc tính top, bottom, left, right để căn chỉnh
- **Fixed** : Dùng để cố định 1 element khi cuộn chuột vẫn hiện thị, nó lấy trình duyệt làm gốc toạ độ ta có thể dùng các thuộc tính top, bottom, left, right để căn chỉnh
- **Sticky** : Dùng để cố định 1 element khi cuộn chuột vẫn hiện thị ta có thể dùng các thuộc tính top, bottom, left, right để căn chỉnh

# Căn giữa trong CSS:

- **text-align**: center; // thuộc tính này có tính kế thừa ví dụ có 1 thẻ div có css text-align : center thì thẻ bên trong nó cũng sẽ kế thừa thuộc tính này của thẻ div

# Flexbox trong CSS:

## Thuộc tính của container

**Các thuộc tính chính của một container flex được liệt kê dưới đây:**

- **flex-direction**: Xác định hướng chính của các flex items. Có thể có các giá trị sau: row (mặc định), row-reverse, column, column-reverse.
- **flex-wrap**: Xác định nếu các flex items nên ngắt dòng hay không. Có thể có các giá trị sau: nowrap (mặc định), wrap, wrap-reverse.
- **justify-content**: Căn chỉnh các flex items theo hướng chính. Có thể có các giá trị sau: flex-start (mặc định), flex-end, center, space-between, space-around, space-evenly.
- **align-items**: Căn chỉnh các flex items theo hướng ngang giao (cross axis). Có thể có các giá trị sau: stretch (mặc định), flex-start, flex-end, center, baseline.
- **align-content**: Căn chỉnh các dòng flex theo hướng ngang giao khi có nhiều hơn một dòng. Có thể có các giá trị sau: stretch (mặc định), flex-start, flex-end, center, space-between, space-around, space-evenly.

## Thuộc tính của flex items

**Các thuộc tính chính của một flex item được liệt kê dưới đây:**

- **order**: Xác định thứ tự hiển thị của flex item trong container. Giá trị mặc định là 0, có thể sử dụng số nguyên dương hoặc âm.
- **flex-grow**: Xác định tỷ lệ mà flex item sẽ phát triển nếu có không gian còn trống trong container. Giá trị mặc định là 0.
- **flex-shrink**: Xác định tỷ lệ mà flex item sẽ thu nhỏ nếu không đủ không gian trong container. Giá trị mặc định là 1.
- **flex-basis**: Xác định kích thước cơ bản của flex item trước khi được phân bổ không gian còn trống. Giá trị mặc định là auto.
- **flex**: Là thuộc tính viết tắt cho flex-grow, flex-shrink, và flex-basis. Ví dụ: flex: 1 1 auto.
- **align-self**: Cho phép ghi đè thuộc tính align-items của container cho một flex item cụ thể. Có thể có các giá trị sau: auto, flex-start, flex-end, center, baseline, stretch.

**Follow link** : https://github.com/vuducthanh0115/note/blob/main/HTML&CSS.md

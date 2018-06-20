# Thực hiện hoạt động ghi đa luồng trong SQLite gần như cùng một thời điểm / Almost row level lock performance. 

Chào các bạn,

Như bạn đã biết, SQLite là cơ sở dữ liệu đơn luồng mặc định được đưa vào hệ điều hành Linux. Có rất nhiều nghiên cứu về sử dụng SQLite để lưu trữ dữ liệu. Có rất nhiều nghiên cứu về cách truy cập cơ sở dữ liệu SQLite cho hoạt động ghi đa luồng. Tôi sẽ chia sẻ nghiên cứu nhỏ của tôi về cách để đạt được hoạt động ghi đa luồng trên cơ sở dữ liệu SQLite.

Chúng ta hãy cùng tìm hiểu ưu điểm và nhược điểm của SQLite
### Ưu điểm.
- SQLite được viết bằng ngôn ngữ lập trình C thuẩn. Vì vậy, nó truy cập vào ổ đĩa hoặc bộ nhớ cơ sở dữ liệu và xử lý dữ liệu là nhanh nhất. Think about if you use SSD disk. 

- SQLite hỗ trợ bộ nhớ trong. Trong bộ nhớ SQLite nhanh gấp gần 2 lần. Nếu bạn hiểu vấn đề phân trang. Nó đủ nhanh.

- SQLite là đơn luồng. Vì vậy, nguy cơ dữ liệu bị hỏng là thấp nhất.

- Cơ sở dữ liệu SQLite nằm trong một file. Vì vậy, chúng tôi có thể di chuyển cơ sở dữ liệu và truy cập bởi một nền tảng khác rất dễ dàng. 

- SQLite is 0 administration for end user. 

- Cross platform. SQLite có thể được sử dụng trên tất cả các nền tảng hệ điều hành chính. 

- OPENSOURCE OPENSOURCE OPENSOURCE !!!

- và vân vân ………..

### Một số nhược điểm.
- Như chúng ta đã đề cập, SQLite là đơn luồng. Vì vậy, nó có nghĩa là SQLite chỉ có thể thực hiện một thao tác ghi tại một thời điểm
- Như chúng ta đã đề cập, SQLite giữ cơ sở dữ liệu trong một file.Vì vậy,có nghĩa là nó khóa toàn bộ cơ sở dữ liêụ trong quá trình ghi. Điều này rất không phù hợp đối với cơ sở dữ liệu truy cập lớn và sâu.. 
- Không yêu cầu xác thực mức ứng dụng
### Thực hiện ghi đa luồng cùng một lúc.
Bây giờ tôi sẽ cố gắng đưa ra mẹo nhỏ để làm cho hoạt động ghi gần như cùng một lúc.

**Chú ý: SQLite không bao giờ cho phép bạn hoàn thành ROW LEVEL LOCK. Vì vậy, không cần phải lãng phí thời gian để tìm kiếm về nó.**

Tất nhiên nó có thể tác động đến hiệu suất nếu bạn thực hiện thao tác ghi trên phần lớn bảng. Nhưng luồng thứ hai sẽ không chờ đợi nhiều thời gian cho đến khi kết thúc hoạt động đầu tiên

![](https://media.licdn.com/dms/image/C5612AQFU1Qb1s5ET0w/article-inline_image-shrink_1000_1488/0?e=2128896000&v=beta&t=l4oDD044Ifz5bnf_9Z0mLqE8H10w5nIFL0AOojqQAw0)

Bời vì như bạn biết index B TREE là siêu nhanh. Và hành động GHI của bạn sẽ thực hiện theo ID của từng dòng đã được index theo mặc định

Ví dụ:

**delete from table where name='Fariz'** // nó sẽ khóa toàn bộ cơ sở dữ liệu chính trong 10 giây..

THAY BẰNG:

**insert into tepm.Lock select rowid,'processid' from table where name = 'Fariz';** // nó sẽ chỉ khóa cơ sở dữ liệu tạm thời trong 10 giây.

**delete from table where ROWID in ( select rowid from tepm.Lock where name='fariz' and processid='XYZ' )** // việc xóa sẽ khóa file cơ sở dữ liệu trong 0,001 giây.

Tôi đã làm xong thử nghiệm nhỏ và kết quả là rất tốt. Tôi đã có thể hoàn thành 2 hoạt động cập nhật tương tự trong 11 giây, mỗi giao dịch mất 10 giây cho bảng bao gồm 40 triệu dòng.

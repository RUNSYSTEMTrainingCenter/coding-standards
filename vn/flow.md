## GMO-Z.com RUNSYSTEM Git flow

Flow tham khảo: [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)

### Giả định
* Đã tạo Central Repository (Nguồn trung tâm) trên Github（hoặc Bitbucket, Gitlab）.
* Branch mặc định của Central Repository là master, code đang chạy trên production.
* Branch phát triển của Central Repository là develop, code có thể chạy được nhưng chưa chạy trên production.
* Lập trình viên có thể  fork (tạo nhánh) đối với Central Repository.
* Đã quyết định người review và người có quyền merge.

### Nguyên tắc
* Mỗi pull-request tương ứng với một ticket.
* Mỗi một pull-request có một hoặc nhiều commit.
* Pull-request title phải đặt sao cho tương ứng với title của task với format `refs [Loại ticket] #[Số ticket] [Nội dung ticket]` （Ví dụ: `refs bug #1234 Sửa lỗi cache`）.
* Đối với commit title, trong trường hợp pull-request đó chỉ có 1 commit thì có thể đặt commit title tương tự như trên là `refs [Loại ticket] #[Số ticket] [Nội dung ticket]` （Ví dụ: `refs bug #1234 Sửa lỗi cache`）.\
  Tuy nhiên với trường hợp 1 pull-request có chứa nhiêù commit thì cần phải ghi rõ trong nội dung commit title là trong commit đó xử lý đối ứng vấn đề gì.
    * Ví dụ:
        1. Pull-request title: `refs bug #1234 Sửa lỗi cache`
        2. Trong trường hợp pull-request có 2 commit thì nội dung commit title của 2 commit sẽ tương ứng như sau
            * `Tạo method thực hiện việc clear cache trong Model`
            * `Tại controller gọi method ở Model để thực hiện việc clear cache`
* Tại môi trường local(trên máy lập trình viên), tuyệt đối không được thay đổi code khi ở branch master.Nhất định phải thao tác trên branch khởi tạo để làm task.

### Chuẩn bị

1. Trên Github (Bitbucket, Gitlab), fork Central Repository về tài khoản của mình（repository ở tài khoản của mình sẽ được gọi là Forked Repository）.

2. Clone (tạo bản sao) Forked Repository ở môi trường local.Tại thời điểm này, Forked Repository sẽ được tự động đăng ký dưới tên là `origin`.
    ```sh
    $ git clone [URL của Forked Repository]
    ```

3. Truy cập vào thư mục đã được tạo ra sau khi clone, đăng ký Central Repository dưới tên `upstream`.
    ```sh
    $ cd [thư mục được tạo ra]
    $ git remote add upstream [URL của Central Repository]
    ```

### Quy trình

Từ đây, Central Repository và Forked Repository sẽ được gọi lần lượt là `upstream` và `origin`.

1. Đồng bộ hóa branch master tại local với upstream.
    ```sh
    $ git checkout master
    $ git pull upstream master
    ```

2. Tạo branch để làm task từ branch master ở local. Tên branch mô tả loại hành động, người thực hiện và task id（Ví dụ: `feature/ThienLV/1234`）.
    ```sh
    $ git checkout master # <--- Không cần thiết nếu đang ở trên branch master
    $ git checkout -b feature/ThienLV/1234
    ```

3. Tiến hành làm task（Có thể commit bao nhiêu tùy ý）.

    3.1 `$ git add .` và `$ git commit -m "title"`
    
    3.2 Khi có yêu cầu từ người review sửa code muốn add thêm vào stage ta có thể dùng `$ git commit --amend --no-edit`
  
4. Push code lên origin.

    4.1 Push lần đầu: 
    
    ```sh
    $ git push origin feature/ThienLV/1234 # Nếu đang ở branch mà muốn push lên thì `git push origin HEAD`
    ```
    
    4.2 Nếu từ bước 6 người reviewer có yêu cầu chính sửa code thì: `f` = force push
    
    ```sh
    $ git push origin feature/ThienLV/1234 -f # Nếu đang ở branch mà muốn push lên thì `git push origin HEAD -f`
    ```
5. Tại origin trên Github（Bitbucket, Gitlab）、từ branch `feature/ThienLV/1234` đã được push lên hãy gửi pull-request đối với branch master của upstream.

6. Hãy gửi link URL của trang pull-request cho reviewer trên chatwork để tiến hành review code.

    6.1. Trong trường hợp reviewer có yêu cầu sửa chữa, hãy thực hiện các bước 3. 〜 5..
    6.2. Tiếp tục gửi lại URL cho reviewer trên chatwork để tiến hành việc review code.

7. Nếu trên 2 người reviewer đồng ý với pull-request, người reviewer cuối cùng sẽ thực hiện việc merge pull-request.
   Revewer xác nhận sự đồng ý bằng comment LGTM.
   
8. Quay trở lại 1.



#### Khi xảy ra conflict trong quá trình rebase

Khi xảy ra conflict trong quá trình rebase, sẽ có hiển thị như dưới đây (tại thời điểm này sẽ bị tự động chuyển về một branch vô danh)
```sh
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: refs #1234 Sửa lỗi cache
Using index info to reconstruct a base tree...
Falling back to patching base and 3-way merge...
Auto-merging path/to/conflicting/file
CONFLICT (add/add): Merge conflict in path/to/conflicting/file
Failed to merge in the changes.
Patch failed at 0001 refs #1234 Sửa lỗi cache
The copy of the patch that failed is found in:
    /path/to/working/dir/.git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```

1. Hãy thực hiện fix lỗi conflict thủ công（những phần được bao bởi <<< và >>> ）.
Trong trường hợp muốn dừng việc rebase lại, hãy dùng lệnh `git rebase --abort`.

2. Khi đã giải quyết được tất cả các conflict, tiếp tục thực hiện việc rebase bằng:

    ```sh
    $ git add .
    $ git rebase --continue
    ```
#### References
- [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
- [Framgia](https://github.com/framgia/coding-standards/blob/master/vn/git/flow.md)

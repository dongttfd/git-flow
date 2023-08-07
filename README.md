# Git flow
# Documentation

* [Nguyên tắc `git flow`](#nguyên-tắc-git-flow)
* [Các câu lệnh cần biết để sử dụng trong bài viết](#các-câu-lệnh-cần-biết-để-sử-dụng-trong-bài-viết)
* [Chương 1: Các khái niệm cần biết cần hiểu](#chương-1-các-khái-niệm-cần-biết-cần-hiểu)
    - [Giải nghĩa một số khái niệm](#giải-nghĩa-một-số-khái-niệm)
        - [`GIT` là gì](#git-là-gì)
        - [`COMMIT` là gì](#commit-là-gì)
            - [Thông tin cơ bản của `commit`](#thông-tin-cơ-bản-commit-cần-biết)
        - [`BRANCH` là gì](#branch-là-gì)
        - [`HEAD` là gì](#head-là-gì)
        - [Thế nào là `BASE`?](#thế-nào-là-base)
        - [`REBASE` là gì](#rebase-là-gì)
        - [`REBASE` khác `MERGE` ở chỗ nào?](#rebase-khác-merge-ở-chỗ-nào)
    - [Quy trình chuẩn để đạt được `nguyên tắc số 2`?](#quy-trình-chuẩn-để-đạt-được-nguyên-tắc-số-2)
    - [So sánh 2 quy trình merge](#so-sánh-2-quy-trình-merge)
        - [Xét các riêng nhánh `develop`](#xét-các-riêng-nhánh-develop)
    - [Sự quan trọng và ý nghĩa của `Nguyên tắc số 2`](#sự-quan-trọng-và-ý-nghĩa-của-nguyên-tắc-số-2)
    - [Câu hỏi, kịch bản thường gặp](#câu-hỏi-kịch-bản-thường-gặp)

* [Chương 2: Quy trình thực tế](#chương-2-quy-trình-thực-tế)
    - [Rebase với nhánh `develop` ở local](#cách-thứ-1-rebase-với-nhánh-develop-ở-local)
    - [Rebase với nhánh `develop` ở repository](#cách-thứ-2-rebase-với-nhánh-develop-ở-repository)
    - [Rebase nhưng không cần commit code](#cách-thứ-3-rebase-nhưng-không-cần-commit-code)

* [Chương 3: Sửa, gộp commit với `--amend` và `rebase`](#chương-3-sửa-gộp-commit-với---amend-và-rebase)
    - [Sửa commit gần nhất với git commit `--amend`](#sửa-commit-gần-nhất-với-git-commit---amend)
    - [Gộp commit với `git rebase`](#gộp-commit-với-git-rebase)


# Nguyên tắc `git flow`
0. Luôn lấy code mới nhất từ nhánh base (`develop`) trước khi tạo `Merge / Pull request` để fix conflict (nếu có) 
1. Tạo merge request chỉ có 1 commit duy nhất
2. **Chỉ** được phép tạo commit merge với nội dung `from <feature/<your-branch> to develop` và không được có commit merge khác
3. Không merge chéo nhánh con

# Các câu lệnh cần biết để sử dụng trong bài viết
```
git commit --amend -m "<your message>" // Sửa lại commit gần nhất
git rebase <your-branch> // rebase nhánh đang đứng dựa trên (base on) <your branch>
git merge <your-branch>// Cần hiểu git merge để phân biệt được rebase
git checkout <branch | filecode | folder code> // chuyển branch, tạo mới branch
git pull // lấy code từ repository
```
# Chương 1: Các khái niệm cần biết cần hiểu
## Giải nghĩa một số khái niệm

### `GIT` là gì?
- `git` là công cụ quản lý phiên bản
### `COMMIT` là gì?
- `commit` được hiểu như một sự cam kết để xuất bản 1 phiên bản của của sản phẩm (ở đây là source code)

#### Thông tin cơ bản `commit` cần biết:
- Mã commit (`hash`) là mã định danh của commit
- Tin nhắn (`message`) là nội dung ngắn của commit
- Các thông tin thay đổi liên quan đến file trong source code, thời gian commit ...
- Thông tin của commit trước đó
- Description là mô tả commit
- ...
### `BRANCH` là gì?
- Nghĩa của `branch` là nhánh, chứa một danh sách các commit liên kết với nhau

### `CONFLICT` là gì?
- Là sự xung đột giữa các phiên bản hay nói khách khác là xung đột giữa các lần commit của bạn với commit khác của bạn, hoặc giữa commit của bạn với commit đồng nghiệp.

### `HEAD` là gì?

Là con trỏ của commit cuối cùng được tham chiếu đến trên nhánh của bạn đang đứng.

Sử dụng: `git log --oneline` sẽ thấy **HEAD** đang tham chiếu đến commit có mã `708272a` của nhánh `feature_1`. Tức là: "Hiện tại tôi đang đứng ở nhánh `feature_1` và commit tôi đang đang tham chiếu đến có mã `708272a`."

```
708272a (HEAD -> feature_1) feature_1 #1
60ed977 (develop) #3
119a5df (master) #2
6264f2d #1
```


### Thế nào là `BASE`?

Giả sử trên nhánh `develop` hiện tại có các commit sau:
```
D---E---F---G---H develop
```
Bạn đang ở nhánh `develop` (có HEAD đang tham chiếu đến H) và thực hiện tạo nhánh mới có tên `feature_2` với lệnh
```
git checkout -b feature_2
```
Kết quả `git flow` sau khi tạo nhánh chúng ta sẽ được `feature_2` có HEAD tham chiếu đến commit `H`
```
                  feature_2 
                 /
D---E---F---G---H develop
```

- Cả 2 nhánh đều có các commit `D` `E` `F` `G` `H` => các commit này được gọi là `BASE`

- Ta cũng thấy rằng HEAD của cả 2 nhánh `develop` và `feature_2` đều tham chiếu đến `H`

- Khi đó ta có thể gọi `develop` là `BASE` của `feature_2`

- Vậy có thể nói, ngay từ khi tạo 1 nhánh mới (nhánh `Y`) từ một nhánh ban đầu (`X`) thì X chính là `BASE` của nhánh Y

**Chú ý**: Nhứng đoạn tiếp theo trong documents này sẽ coi `develop` là base của toàn project

Ngoài ra chúng ta thường sẽ gặp những kiểu branch có dạng:

- Ví dụ 1: *`develop` là `BASE` của nhiều nhánh*
```
                   A---B---C feature_2 
                 /
D---E---F---G---H develop
                 \
                   X---Y---Z feature_3 
```
- Ví dụ 2: *Một phần `develop` là `BASE` của nhiều nhánh*

```
                   A---B---C feature_2 
                 /
D---E---F---G---H---I---J develop
                 \
                   X---Y---Z feature_3 
```
- Ví dụ 3: *`develop` là `BASE` của `feature_2` nhưng không là `BASE` của `feature_3`*

```
                             A---B---C feature_2 
                            /
D---E---F---G---H---I---J---M develop
                 \         /
                  X---Y---Z feature_3
```

**Vậy: `BASE` là phần gốc tính từ điểm xuất phát (commit đầu tiên của nhánh chính - `develop`) đến điểm bắt đầu rẽ nhánh (commit đầu tiên của `branch`)**

---
## `REBASE` là gì?
**`REBASE` là quá trình đặt lại gốc của 1 nhánh hay nói cách khác là quá trình sửa lại vị trí commit mà `branch` đó bắt đầu rẽ nhánh**

Ví dụ `git flow` dưới đây
```
                   A---B---C feature_2 
                 /
D---E---F---G---H---I---J develop
                 \
                   X---Y---Z feature_3
```
Trên ví dụ 2 ta thấy trên `feature_2` và `feature_3` không có đầy đủ commit của nhánh `develop`. Chúng ta phải làm sao cho 2 commit `I`, `J` trên `develop` đều thuộc nhánh `feature_2` và `feature_3` => quá trình này gọi là `REBASE`

Rebase nhánh `feature_2` bằng các câu lệnh
```
git checkout feature_2 // chuyển HEAD sang nhánh feature_2
git rebase develop // rebase nhánh feature_2 bằng nhánh develop
```
Kết quả
```
                          A---B---C feature_2 
                         /
D---E---F---G---H---I---J develop
                 \
                   X---Y---Z feature_3
```

Tương tự khi rebase nhánh `feature_3`
```
                          A---B---C feature_2 
                         /
D---E---F---G---H---I---J develop
                         \
                          X---Y---Z feature_3
```
---
## `REBASE` khác `MERGE` ở chỗ nào?
Merge sẽ luôn tạo ra commit merge
Ví dụ `git flow` dưới đây
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J develop
                 \ 
                  X---Y feature_3
```

**Nếu thực hiện `merge` nhánh `develop` vào nhánh `feature_3`**
```
git checkout feature_3 // chuyển HEAD sang nhánh feature_3
git merge develop
```
Kết quả
```
                  A---B---C feature_2 
                 /
D---E---F---G---H---I---J develop
                 \       \
                  X---Y---M feature_3
```
***Nhận thấy:***
Khi `MERGE` nhánh `develop` vào nhánh `feature_3` sẽ tự tạo ra commit merge `M` với nội dung có dạng `Merge from develop to feature_3 ...`

Trong quá trình làm việc `develop` luôn là nhánh chung, vậy khi merge `feature_3` vào develop:
```
git checkout develop // chuyển HEAD sang nhánh develop
git merge feature_3
```
Quá trình merge của git
```
// step 1
                  A---B---C feature_2 
                 /
D---E---F---G---H---I---J---(M1) develop
                 \         /
                  X---Y---M feature_3

=========================================
// step 2
                  A---B---C feature_2 
                 /
D---E---F---G---H---I---J   develop
                 \       \ /
                  X---Y---M feature_3

// kết quả
                  A---B---C feature_2 
                 /
D---E---F---G---H---I---J---M develop
                 \        /  \
                  X------Y    feature_3
                   
```

Khi `MERGE` nhánh `feature_3` vào nhánh `develop`, do commit `M` là commit merge của `J` và `Y`, theo logic merge thì sẽ tạo ra commit merge (`M1`) giữa `J` và `M` nhưng `M` đã bao gồm `J` trước đó nên HEAD của develop sẽ không tạo ra `M1` trỏ vào `M`

**Tuy nhiên**
- Việc giữ nguyên `M` `Nguyên tắc số 2` ban đầu đề ra khi merge `feature_3`
- Vấn đề sẽ xảy ra càng nhiều commit dạng `M` khi chúng ta merge nhiều nhánh theo cách này
- Sẽ càng nan giải hơn khi chúng ta merge trực tiếp từ `repository` vì khi đó git đang hiểu nhánh ở `repository` là 1 nhánh khác nhánh hiện tại kiểu như: `origin/develop` là một nhánh khác với `develop` ở local hoặc `origin/feature_2` là một nhánh khác với `feature_2` local
- Sẽ càng nan giải hơn nữa nữa nếu chúng ta merge chéo nhánh từ `repository` về local như `origin/develop` về `feature_2` local

Kết quả khi merge thêm `feature_2` vào develop với cách merge trên
```
// git checkout feature_2 && git merge develop

                  A---B---C---M2 feature_2 
                 /           /
D---E---F---G---H---I---J---M develop
                 \        /  \
                  X------Y    feature_3

// git checkout develop && git merge feature_2

                  A-----B-----C    feature_2
                 /             \ /
D---E---F---G---H---I---J---M---M2 develop
                 \        /  \
                  X------Y    feature_3

                                                      feature_2 
                                                     /
D---E---F---G---H---I---J---X---Y---M---A---B---C---M2 develop
                                     \
                                       feature_3
```
`M` và `M2` ở trên có dạng: `Merge from develop to feature...`

---
## **Quy trình chuẩn để đạt được nguyên tắc số 2**
Với ví dụ dưới đây, ta cần merge sao cho các commit của `feature_3` được merge vào nhánh develop, và không vi phạm `Nguyên tắc số 2`
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J develop
                 \ 
                  X---Y feature_3
```

- Bước 1: chuyển sang nhánh `feature_3` và `rebase` nhánh `develop`
```
git checkout feature_3 // chuyển HEAD sang nhánh feature_3
git rebase develop // rebase nhánh feature_3 bằng nhánh develop
```
Trạng thái sau bước 1
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J develop
                         \ 
                          X---Y feature_3
```
- Bước 2: chuyển sang nhánh `develop` và `merge` nhánh `feature_3`
```
git checkout develop // chuyển HEAD sang nhánh develop
git merge feature_3 // merge nhánh feature_3 vào develop
```
Trạng thái sau bước 2
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J------M3 develop
                         \     /
                          X---Y feature_3

```

Vậy khi merge với quy trình này, chúng ta chỉ phải tạo ra 1 commit merge duy nhất là `M3` có dạng `Merge from feature_3 to develop`

Tương tự quy trình trên với cách merge này sử dụng cho việc merge `feature_2` vào `develop` ta được kết quả:
```
                                 A---B feature_2 
                                /     \
D---E---F---G---H---I---J------M3-----M2 develop
                         \     /
                          X---Y feature_3
```
---
## So sánh 2 quy trình merge
Trở lại ví dụ trên với git flow ban đầu
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J develop
                 \ 
                  X---Y feature_3
```

Merge khi dùng quy trình `REBASE`
```
                                 A---B feature_2 
                                /     \
D---E---F---G---H---I---J------M3-----M2 develop
                         \     /
                          X---Y feature_3
```

Merge không dùng quy trình `REBASE`
```
                  A-----B-----C    feature_2
                 /             \ /
D---E---F---G---H---I---J---M---M2 develop
                 \        /  \
                  X------Y    feature_3
```

### Xét các riêng nhánh `develop`

Thoạt nhìn thì các bạn thấy kết quả giống nhau nhưng không phải vậy, nó khác nhau ở các điểm sau:
- Thứ tự commit của nhánh không phải base luôn là commit mới nhất
- Nhánh feature luôn có `BASE` là `develop` mà không phải tạo bất kì commit merge nào dạng `from develop to feature...`

**Hai điều trên sẽ càng thấy rõ hơn với trường hợp chúng ta làm nhiều nhánh hơn**
Git flow với 3 features theo thứ tự merge về `develop` các nhánh: `feature_4`, `feature_3`, `feature_2`
```
                   A---B feature_2 
                 /
D---E---F---G---H---I---J develop
         \       \ 
          \       X---Y feature_3
           \ 
            Q---R feature_4

```

Merge khi dùng quy trình `REBASE`
```
                                               A---B feature_2 
                                              /     \
D---E---F---G---H---I---J-----------M4-------M3------M2 develop
                         \         /  \     /
                          \       /    X---Y feature_3
                           \     /        
                            Q---R feature_4
```

Merge không dùng quy trình `REBASE`
```

                   A-----------------------B  
                  /            feature_4    \   feature_2
                 /            /              \ /
D---E---F---G---H---I---J---M4---M3----------M2 develop
         \       \         /    / 
          \        X------/----Y feature_3
           \             /
            Q-----------R
```

**Trong quá trình làm việc chúng ta thường xuyên làm những điều sau**
1. `feature_2` có commit `A`
2. `develop` đã có người khác merge `feature_x` vào gồm nhóm commit `X` và commit merge `MX`
3. Merge `develop` về `feature_2` (tạo ra commit `MXA`) xong làm tiếp commit `B`
4. `develop` đã có người khác merge `feature_y` vào gồm nhóm commit `Y` và commit merge `MY`
5. Merge lần 2 `develop` về `feature_2` (tạo ra commit `MYB`) xong làm tiếp commit `C`
6. Merge `feature_2` vào `develop` tạo ra commit `MC`

---
Luồng git flow khi không dùng `REBASE`
```
#1 feature_2 có commit A

                   A feature_2 
                 /
D---E---F---G---H develop

=======================================================================================

#2 develop đã có người khác merge feature_x vào gồm nhóm commit X và commit merge MX

                   A feature_2 
                 /
D---E---F---G---H---MX develop
                 \ /  \
                  X    feature_x

=======================================================================================

#3 Merge develop về feature_2 (commit merge MXA) xong làm tiếp commit B

                  A---MXA---B feature_2 
                 /   /
D---E---F---G---H---MX develop
                 \ /  \
                  X    feature_x

=======================================================================================

#4 develop đã có người khác merge feature_y vào gồm nhóm commit Y và commit merge MY

                  A---MXA---B feature_2 
                 /   /
D---E---F---G---H---MX---MY develop
                 \ /  \ / \
                  X    Y   feature_y

=======================================================================================

#5 Merge lần 2 develop về feature_2 (commit merge MYB) xong làm tiếp commit C

                  A---MXA---B---MYB---C feature_2 
                 /   /         /
D---E---F---G---H---MX-------MY develop
                 \ /  \ /     \
                  X    Y       feature_y

=======================================================================================

#6 Merge feature_2 vào develop tạo ra commit MC

                  A---MXA---B---MYB---C    feature_2 
                 /   /         /       \  /
D---E---F---G---H---MX-------MY---------MC develop
                 \ /  \ /
                  X    Y       

```

**Điều này cực kì tai hại khi các bạn thấy nhánh `feature_2` mà bạn đang làm có lẫn lộn các commit của mình và develop + các nhánh khác lẫn lộn nhau**

**Bạn cũng không thể xác định được `BASE` của nhánh của mình bắt đầu từ đâu vì commit đang lẫn lộn**

**Các bạn cũng chẳng thể gộp được commit `A`, `B`, `C` nếu có yêu cầu gộp**

---
Luồng git flow khi dùng `REBASE`
```
#1 feature_2 có commit A

                   A feature_2 
                 /
D---E---F---G---H develop

=======================================================================================

#2 develop đã có người khác merge feature_x vào gồm nhóm commit X và commit merge MX

                   A feature_2 
                 /
D---E---F---G---H---MX develop
                 \ / 
                  X feature_x

=======================================================================================

#3 Merge develop về feature_2 sử dụng REBASE xong làm tiếp commit B 

                      A---B feature_2 
                     /
D---E---F---G---H---MX develop
                 \ / 
                  X feature_x

=======================================================================================

#4 develop đã có người khác merge feature_y vào gồm nhóm commit Y và commit merge MY

                      A---B feature_2 
                     /
D---E---F---G---H---MX---MY develop
                 \ /  \ /
                  X    Y feature_y

=======================================================================================

#5 Merge lần 2 develop về feature_2 sử dụng REBASE xong làm tiếp commit C

                            A---B---C feature_2 
                           /         
D---E---F---G---H---MX---MY develop
                 \ /  \ /
                  X    Y feature_y

=======================================================================================

#6 Merge feature_2 vào develop tạo ra commit MC

                            A---B---C feature_2 
                           /         \
D---E---F---G---H---MX---MY----------MC develop
                 \ /  \ / 
                  X    Y

```

**Bạn thấy nhánh `feature_2` của mình luôn có `BASE` theo `develop` không bị xen kẽ với commit với nhánh `feature_2`**
**Những commit merge (`MXA`, `MYB`) hoàn toàn không tồn tại**
**Bạn sẽ không gặp vấn đề khi gộp commit `A`, `B`, `C`**
## Sự quan trọng và ý nghĩa của `Nguyên tắc số 2`
- Nguyên tắc số 2 bản chất là hệ quả của quy trình Merge kết hợp rebase
- Nếu bạn tuân thủ nó bạn sẽ hạn chế được commit merge không cần thiết (tôi gọi là commit rác)
- Sẽ dễ dàng hơn cho bạn khi muốn gộp hoặc sửa commit

---
## Câu hỏi, kịch bản thường gặp
**Question:** Tôi không muốn lấy `develop` về trước khi tạo request merge được không, vì khi đó tôi cũng chẳng có commit merge nào từ develop về nhánh của tôi cả?

**Answer:** Tất nhiên là được, miễn sao bạn không bị conflict với nhánh develop, và chức năng của bạn không ảnh hưởng bởi code `develop` và ngược lại. Nhưng tôi khuyên bạn nên làm điều này.

---

**Question:** Tôi có thể pull trực tiếp từ `develop` trên `repository` về mà vẫn đảm bảo rebase được hay không, nếu có thì bằng cách nào?

**Answer:** Hoàn toàn được, có 2 cách rebase bạn đang hỏi là cách số 2. Hãy chuyển sang nhánh của bạn sử dụng câu lệnh `git pull origin develop --rebase` nếu `origin` là `alias` của `repository` mà bạn đang `remote` tới.

---

**Question:** Tôi cùng làm việc trên 2 máy tính và cùng 1 `branch`, máy 1  tôi `push` code lên, máy 2 pull về dù không có conflict mà nó vẫn tạo ra commit merge dạng như `from origin <my-branch> to <my-branch>`.
Tại sao lại như vậy? Commit này có vi phạm `nguyên tắc số 2` hay không? Nếu vi phạm, Tôi phải làm gì để tránh điều này?

**Answer:**
Tại vì git đang hiểu `origin/<your-branch>` là 1 nhánh khác `your-branch` trên local của bạn nên khi `pull` nó sẽ merge với nhánh local nó sẽ tạo ra commit merge khi cả `origin/<your-branch>` và `your-branch` trên local bị thay đổi, vì bản chất của `pull` là `merge`
Điều này vi phạm `nguyên tắc số 2` ở đầu bài viết
Để tránh điều này bạn hãy sử dụng thêm `--rebase` khi `pull`


---
**Question:** Tôi sử dụng `git pull --rebase` hoặc `git rebase` trong 1 số trường hợp tôi `push` lên lại bị cancel không cho `push` là do làm sao? Giải quyết thế nào? Tôi có nên `pull` lại nhánh đó không?

**Answer:** Khi bạn sử dụng `rebase` trong một số trường hợp kể cả không có conflict (git sẽ auto merge file) thì commit mà `HEAD` của bạn đang trỏ tới (commit gần) sẽ bị thay đổi => mã `hash` của commit thay đổi sẽ khác với commit lần trước của bạn đã push lên. Bạn không nên pull lại lần nữa vì code của bạn ở repository đã có ở local rồi, nếu bạn pull về sẽ phải fix conflict 1 lần nữa. Bạn hãy sử dụng push với `--forge` hoặc `-f` với câu lệnh push của bạn


---
**Question:** Trong một số trường hợp rebase tôi bị `conflict` phải làm thế nào?

**Answer:** Câu trả lời ở chương tiếp theo

# Chương 2: Quy trình thực tế

Để thực hiện quy trình merge có `rebase` ta sử dụng 1 trong 2 cách ở dưới.

Các thông số cho ví dụ:

// Alias repository remoted

`origin: git@gitlab.com....`

// base branch

`develop`

// your branch

`feature_x`

## Cách thứ 1: Rebase với nhánh `develop` ở local
- ***Bước 0:*** commit code đã thay đổi của bạn và push lên nhánh của bạn và `push` lên nhánh của bạn trên repository (`repository/<your-feature>`);
```

git add <source-changed> // add file changed

git commit -m "<your-message>" // your commit

git push origin feature_x // origin: là alias của repository mà bạn remote tới; feature_x là nhánh của bạn 

```

- ***Bước 1:*** Chuyển sang nhánh `develop` và `pull` code mới nhất về
```
git checkout develop

git pull // hoặc git pull origin develop
```

- ***Bước 2:*** Chuyển lại nhánh của bạn và thực hiện rebase

```
git checkout feature_x

git rebase develop
```

**Nếu không có gì thay đổi (`Already up to date.`):** thực hiện `Bước 3`

**Nếu có conflict xảy ra** thực hiện `Bước 2.1`

**Không có conflict xảy ra** thực hiện `Bước 3`

- ***Bước 2.1:*** Xử lý conflict

Thực hiện fix conflict những file có trạng thái màu đỏ: `both mergeed`, `both added`, `both deleted` ... sau đó add file
```
git add <source-conflicted>
```

Gõ lệnh sau để tiếp tục rebase
```
git rebase --continue
```
*Có thể git mở trình editor lên bắt sửa file commit của bạn, nếu bạn cần thay đổi message của commit cuối cùng thì sửa không thì thoát*

Sau khi `continue` vẫn còn conflict thì lặp lại `Bước 2.1`

***Nếu gặp confict mà không muốn rebase nữa thì `abort`***
```
git rebase --abort
```
- ***Step 3:*** `push` lại code lên repository nếu có sự thay đổi
```
git push / hoặc: git push origin feature_x
```

Nếu không cho push hãy thêm `--forge` hoặc `-f` vào câu lệnh `push`

**Tạo `Merge / Pull request nếu cần merge lại develop`**

---
## Cách thứ 2: Rebase với nhánh `develop` ở repository

Cách chỉ khác `Cách thứ 1` ở chỗ `Bước 1` và `Bước 2` gộp lại làm 1 bước

- ***Bước 0:*** commit code đã thay đổi của bạn và push lên nhánh của bạn và `push` lên nhánh của bạn trên repository (`repository/<your-feature>`);
```

git add <source-changed> // add file changed

git commit -m "<your-message>" // your commit

git push origin feature_x // origin: là alias của repository mà bạn remote tới; feature_x là nhánh của bạn 

```

- ***Bước 1:*** Thực hiện pull với rebase tại nhánh `feature_x`

```
git pull --rebase origin  develop
```

**Nếu không có gì thay đổi (`Already up to date.`):** thực hiện `Bước 2`

**Nếu có conflict xảy ra** thực hiện `Bước 1.1`

**Không có conflict xảy ra** thực hiện `Bước 2`

- ***Bước 1.1:*** Xử lý conflict

Thực hiện fix conflict những file có trạng thái màu đỏ: `both mergeed`, `both added`, `both deleted` ... sau đó add file
```
git add <source-conflicted>
```

Gõ lệnh sau để tiếp tục rebase
```
git rebase --continue
```
*Có thể git mở trình editor lên bắt sửa file commit của bạn, nếu bạn cần thay đổi message của commit cuối cùng thì sửa không thì thoát*

Sau khi `continue` vẫn còn conflict thì lặp lại `Bước 1.1`

***Nếu gặp confict mà không muốn rebase nữa thì `abort`***
```
git rebase --abort
```
- ***Bước 2:*** `push` lại code lên repository nếu có sự thay đổi
```
git push / hoặc: git push origin feature_x
```

Nếu không cho push hãy thêm `--forge` hoặc `-f` vào câu lệnh `push`

---
## Cách thứ 3: Rebase nhưng không cần commit code

Cách này giúp bạn không cần commit code hiện tại mà vẫn có thể `rebase`, sử dụng khi bạn chưa muốn commit hoặc push code của mình lên.

Tuy nhiên Bạn phải đảm bảo 2 điều kiện:

1. Code của bạn không có file thêm mới (untracked files)
2. Nếu code của bạn có file thêm mới thì nhánh `develop` không được có file đó.
3. Bạn không xoá hoặc thay đổi tên của 1 file nào mà trên develop có commit thay đổi nó kể từ `BASE` lần trước.
4. Các file không thể `stash` khác

- ***Bước 0:*** Lưu code đã thay đổi vào `stash`

```
git stash
```

- ***Bước 1:*** Thực hiện pull với rebase tại nhánh của bạn (`feature_x`)
```
git pull --rebase origin  develop
```

**Nếu không có gì thay đổi (`Already up to date.`):** thực hiện `Bước 2`

**Nếu có conflict xảy ra** thực hiện `Bước 1.1`

**Không có conflict xảy ra** thực hiện `Bước 2`

- ***Bước 1.1:*** Xử lý conflict

Thực hiện fix conflict những file có trạng thái màu đỏ: `both mergeed`, `both added`, `both deleted` ... sau đó add file
```
git add <source-conflicted>
```

Gõ lệnh sau để tiếp tục rebase
```
git rebase --continue
```
*Có thể git mở trình editor lên bắt sửa file commit của bạn, nếu bạn cần thay đổi message của commit cuối cùng thì sửa không thì thoát*

Sau khi `continue` vẫn còn conflict thì lặp lại `Bước 1.1`

***Nếu gặp confict mà không muốn rebase nữa thì `abort`***
```
git rebase --abort
```
- ***Bước 2:*** Lấy lại code từ `stash` và code tiếp

Gõ lệnh:

```
git stash pop
```

**Không có conflict xảy ra** Code tiếp

**Nếu có conflict xảy ra** thực hiện `Bước 2.1`

- ***Bước 2.1:*** Bản chất là merge code develop với `stash` gặp conflcit
Thực hiện fix conflict với các file `both edited`, `both added`, `both deleted` ... sau đó add lại từng file bị conflict
```
git add <files>
```
Thực hiện reset tất những files đang ở trạng thái đã add (`added`) về trạng thái trước khi rebase rồi code tiếp
```
git reset .
```

# Chương 3: Sửa, gộp commit với `--amend` và `rebase`
## Sửa commit gần nhất với `git commit --amend`
Dùng khi bạn muốn sửa commit gần nhất: thêm sự thay đổi hoặc sửa message, 

***Thêm sự thay đổi vào commit gần nhất:***

Thêm sự thay đổi
```
git add <files>
```
Thêm vào commit gần nhất
```
git commit --amend
```

git sẽ mở ra editor dạng như sau:

```
last commit of feature 2

> Please enter the commit message for your changes. Lines starting
> with '>' will be ignored, and an empty message aborts the commit.
>
> Date:      Wed Jun 16 02:02:28 2021 +0700
>
> On branch feature_2
> Changes to be committed:
>       modified:   README.md
>
> Changes not staged for commit:
>       modified:   README.md
>

```

Sửa message (dòng đầu tiên) rồi lưu file để kết thúc việc sửa commit

*Trước khi commit `--amend`*

```
A---B---C feature_2 
```

*Sau khi commit `--amend`*

```
A---B---D feature_2 
```

=> commit `C` sẽ được thay bằng commit `D`

---

**Sử dụng `git commit --amend -m "<nội dung message>"` nếu muốn điền luôn message.**

**Nếu xảy ra lỗi check `git log` để xem commit gần nhất đã đổi hay chưa, nếu chưa thì thực hiện lại quy trình amend**

**Nếu bạn thêm bất kì thay đổi gì (không gõ `git add`) mà gõ lệnh `git commit --amend` nó sẽ chỉ sửa message của commit gần nhất.**

**Khi commit với `--amend` mã `hash` của commit đã bị thay đổi, nên nếu commit trước khi sửa đó đã có ở trên `repository` thì bạn phải `push` với `--forge` hoặc `-f`**


## Gộp commit với `git rebase`

Để gộp commit bạn phải đảm bảo không có file nào ở trạng thái màu đỏ trên nhánh của bạn (`untracked`, `modifed`, `deleted` ...)

Giả sử có git flow bên dưới

```
                      A---B feature_2 
                     /
D---E---F---G---H---MX---MY develop
```

Trước khi tạo `Merge / Pull request` tôi được yêu cầu gộp 2 commit `A`, `B` lại thành 1 commit duy nhất là `AB` => tôi sử dụng `rebase` như sau:

Vì số lượng commit hiện tại cần rebase là 2 nên thực hiện gõ lệnh

```
git rebase -i HEAD~2

```

Giải nghĩa:

- `-i` để biểu thị người dùng sẽ chỉnh sửa các commits
- `HEAD~2` tính từ HEAD hiện lại tới 2 commit trước đó


Sau khi gõ lệnh trên sẽ hiển thị dạng như sau:
```
pick f6d8d04 A
pick 8d8a5ca B

> Rebase 6715b0b..8d8a5ca onto 6715b0b (2 commands)
>
> Commands:
> p, pick <commit> = use commit
> r, reword <commit> = use commit, but edit the commit message
> e, edit <commit> = use commit, but stop for amending
> s, squash <commit> = use commit, but meld into previous commit
> f, fixup <commit> = like "squash", but discard this commit's log message
> x, exec <command> = run command (the rest of the line) using shell
> b, break = stop here (continue rebase later with 'git rebase --continue')
> d, drop <commit> = remove commit
> l, label <label> = label current HEAD with a name
> t, reset <label> = reset HEAD to a label
> m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
> .       create a merge commit using the original merge commit's
> .       message (or the oneline, if no original merge commit was
> .       specified). Use -c <commit> to reword the commit message.
>
> These lines can be re-ordered; they are executed from top to bottom.
>
```

ở dòng `Commands` chúng ta thấy có hướng dẫn đầy đủ cho phép chúng ta thực hiện hành động với từng commit. Mục đích của chúng ta ở đây là gộp commit `B` vào commit `A` nên chúng ta sẽ sửa các dòng:

- Sử dụng `squash` hoặc `s` với `B` để các thay đổi của `B` được thêm vào `A` (commit trước đó)
- Sử dụng `pick` với `A` để giữ nguyên A
```
pick f6d8d04 A
squash 8d8a5ca B

> Rebase 6715b0b..8d8a5ca onto 6715b0b (2 commands)
>
> Commands:
> p, pick <commit> = use commit
> r, reword <commit> = use commit, but edit the commit message
> e, edit <commit> = use commit, but stop for amending
> s, squash <commit> = use commit, but meld into previous commit
> f, fixup <commit> = like "squash", but discard this commit's log message
> x, exec <command> = run command (the rest of the line) using shell
> b, break = stop here (continue rebase later with 'git rebase --continue')
> d, drop <commit> = remove commit
> l, label <label> = label current HEAD with a name
> t, reset <label> = reset HEAD to a label
> m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
> .       create a merge commit using the original merge commit's
> .       message (or the oneline, if no original merge commit was
> .       specified). Use -c <commit> to reword the commit message.
>
> These lines can be re-ordered; they are executed from top to bottom.
>
```

Sau khi lưu file sẽ ra màn hình giúp bạn sửa nội dung commit

```
> This is a combination of 2 commits.
> This is the 1st commit message:

A

> This is the commit message #2:

B
```

**Để lại dòng message `A` nếu bạn không muốn giữ message của commit được gộp là `A`**

**Để lại dòng message `B` nếu bạn không muốn giữ message của commit được gộp là `B`**

**Nếu chỉ muốn commit gộp có một message có nội dung `AB` thì xoá 1 trong 2 dòng `A`, `B` rồi sửa dòng còn lại thành `AB`**

```
> This is a combination of 2 commits.
> This is the 1st commit message:

AB

> This is the commit message #2:

```

Lưu file và thoát sẽ được kết quả

```
                      AB feature_2 
                     /
D---E---F---G---H---MX---MY develop
```

**Chú ý**
- Các file log của trong document này sử dụng `terminal` (MACOS) hoặc `GitBash` (Windows), cú pháp mặc định của nó sử dụng tool `vim`
- Không chỉnh sửa các commit merge bằng `rebase`
- Không được gộp commit đã merge vào nhánh `BASE`, ở đây là `develop`
- Không nên xoá commit nếu không cần thiết

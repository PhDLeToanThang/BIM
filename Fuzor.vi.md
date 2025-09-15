# 1. Các mô hình triển khai Fuzor Server?

>>>FAQ1. Hãy phân tích và cho tôi biết về các mô hình triển khai hiện nay của Fuzor Server 2017, 2018, 2019, 2025 trên OS Windows 2019 DC license ? ví dụ: Standard, Cluster Windows 2 nodes ...
và cả thông tin cấu hình kích thước phù hợp cho vCPU, vRAM, vGPU, Virtual disk?

---

Fuzor, một phần mềm VDC (Virtual Design & Construction) được sử dụng rộng rãi trong ngành AEC (Architecture, Engineering, and Construction), chủ yếu hoạt động như một ứng dụng khách (client) với các yêu cầu cao về đồ họa để thực hiện mô phỏng 4D/5D và trải nghiệm VR/AR. Tuy nhiên, Fuzor cũng có các thành phần "Server" chính là Fuzor Network License Server và Network Collaboration Server, đóng vai trò quản lý bản quyền và điều phối các phiên làm việc cộng tác.

Dựa trên thông tin hiện có, Fuzor Server không được thiết kế với các mô hình triển khai phức tạp như các ứng dụng doanh nghiệp lớn có khả năng mở rộng (scale-out) riêng biệt. Thay vào đó, các thành phần máy chủ của Fuzor thường được triển khai dưới dạng một phiên bản độc lập (Standard) trên một máy chủ Windows Server. Đối với nhu cầu sẵn sàng cao (High Availability - HA), việc triển khai dạng cụm (Cluster Windows 2 nodes) sẽ được áp dụng ở cấp độ máy chủ ảo hoặc hệ điều hành Windows Server, chứ không phải là Fuzor Server có cơ chế clustering nội bộ.

Dưới đây là phân tích về các mô hình triển khai và thông tin cấu hình phù hợp cho Fuzor Network License Server và Network Collaboration Server trên hệ điều hành Windows Server 2019 Datacenter Edition.

### Các Mô Hình Triển Khai

1.  **Mô hình Standard (Độc lập)**
    Đây là mô hình triển khai phổ biến nhất, trong đó Fuzor Network License Server và/hoặc Network Collaboration Server được cài đặt trên một máy chủ Windows Server 2019 DC duy nhất. Máy chủ này sẽ chịu trách nhiệm quản lý và phân phối license cũng như điều phối các phiên làm việc cộng tác.
    *   **Ưu điểm:** Đơn giản để cài đặt và quản lý, chi phí thấp.
    *   **Nhược điểm:** Điểm lỗi đơn (Single Point of Failure - SPoF). Nếu máy chủ gặp sự cố, dịch vụ Fuzor sẽ bị gián đoạn.

2.  **Mô hình Cluster Windows 2 nodes (Sẵn sàng cao ở cấp độ hạ tầng ảo hóa)**
    Fuzor Server không có khả năng clustering nội bộ để đạt được sẵn sàng cao. Tuy nhiên, sẵn sàng cao có thể được đạt được bằng cách triển khai máy chủ ảo (Virtual Machine - VM) chạy Fuzor Server trên một cụm Windows Server Failover Cluster (WSFC) gồm 2 node. Trong mô hình này:
    *   Mỗi node vật lý trong cụm WSFC sẽ chạy Windows Server 2019 Datacenter Edition.
    *   Fuzor Network License Server và/hoặc Network Collaboration Server sẽ được cài đặt trên một VM.
    *   VM này sau đó được cấu hình thành một "Highly Available VM" trong cụm Hyper-V hoặc VMware.
    *   Trong trường hợp một node vật lý gặp sự cố, VM chạy Fuzor Server sẽ tự động chuyển sang node vật lý còn lại trong cụm mà không làm gián đoạn đáng kể dịch vụ cho người dùng cuối (trừ một khoảng downtime ngắn khi chuyển đổi).
    *   **Ưu điểm:** Cung cấp khả năng sẵn sàng cao cho dịch vụ Fuzor Server bằng cách bảo vệ VM khỏi lỗi phần cứng của máy chủ vật lý.
    *   **Nhược điểm:** Phức tạp hơn trong cài đặt và quản lý, yêu cầu thêm tài nguyên phần cứng (ít nhất 2 máy chủ vật lý) và phần mềm (Windows Server 2019 Datacenter licenses, Hyper-V hoặc VMware vSphere).

### Cấu hình kích thước phù hợp (vCPU, vRAM, vGPU, Virtual disk)

Các yêu cầu về phần cứng cho Fuzor Server (License và Collaboration) thường không cao như Fuzor Plugin client, vốn đòi hỏi GPU mạnh mẽ cho việc xử lý đồ họa thời gian thực. Fuzor Server chủ yếu xử lý các tác vụ quản lý bản quyền, xác thực và điều phối dữ liệu nhỏ cho các phiên cộng tác. Do đó, trọng tâm cấu hình sẽ là sự ổn định và khả năng xử lý các tác vụ cơ bản của máy chủ. Các phiên bản Fuzor Server 2017, 2018, 2019 và 2025 có thể có những cải tiến về hiệu suất hoặc tính năng, nhưng yêu cầu phần cứng cơ bản cho các thành phần server có xu hướng ít thay đổi đáng kể.

Dưới đây là thông tin cấu hình đề xuất khi triển khai Fuzor Server trên Windows Server 2019 Datacenter Edition dưới dạng máy ảo:

#### Cấu hình cho Windows Server 2019 Datacenter Edition (Hệ điều hành cơ bản)

*   **vCPU:** 2 vCPU.
*   **vRAM:** 2 GB (tối thiểu) đến 4 GB (khuyến nghị cho cài đặt có GUI - Server with Desktop Experience). Đối với các phiên bản Windows Server Core (không có GUI), yêu cầu RAM tối thiểu là 512 MB.
*   **Virtual disk:**
    *   Tối thiểu: 36 GB.
    *   Khuyến nghị: 100 GB hoặc hơn. Nên sử dụng SSD cho hiệu suất tốt hơn.

#### Cấu hình bổ sung cho Fuzor Network License Server / Network Collaboration Server

Vì các thành phần server của Fuzor không yêu cầu xử lý đồ họa nặng, các tài nguyên sau đây sẽ tập trung vào việc đảm bảo hoạt động ổn định và khả năng phục vụ số lượng người dùng đồng thời:

*   **vCPU:**
    *   **Standard (Độc lập):** 2-4 vCPU. Mặc dù 2 vCPU có thể đủ cho Windows Server, việc thêm 1-2 vCPU sẽ cung cấp hiệu suất tốt hơn cho các tác vụ của Fuzor Server và các dịch vụ nền khác.
    *   **Cluster Windows 2 nodes:** 2-4 vCPU cho mỗi VM Fuzor Server.
*   **vRAM:**
    *   **Standard (Độc lập):** 8 GB hoặc hơn. Mặc dù Fuzor Server có thể không tiêu thụ nhiều RAM, việc có đủ bộ nhớ sẽ đảm bảo Windows Server và các dịch vụ chạy mượt mà, đặc biệt khi có nhiều người dùng Fuzor client kết nối hoặc khi server xử lý các tác vụ license/collaboration phức tạp.
    *   **Cluster Windows 2 nodes:** 8 GB hoặc hơn cho mỗi VM Fuzor Server.
*   **vGPU:**
    *   **Không bắt buộc.** Fuzor Server không yêu cầu vGPU vì nó không thực hiện các tác vụ kết xuất đồ họa hoặc mô phỏng 3D nặng.
    *   Yêu cầu GPU mạnh mẽ là dành cho các máy trạm (client) chạy ứng dụng Fuzor để thực hiện trực quan hóa, VR/AR, 4D/5D.
*   **Virtual disk:**
    *   **Standard (Độc lập):** 120 GB SSD. Bao gồm không gian cho hệ điều hành, cài đặt Fuzor Server và dữ liệu liên quan.
    *   **Cluster Windows 2 nodes:** 120 GB SSD. Đối với các cụm, đảm bảo rằng ổ đĩa ảo được lưu trữ trên bộ nhớ dùng chung (shared storage) có hiệu suất cao để hỗ trợ việc chuyển đổi giữa các node.

**Lưu ý quan trọng:**

*   **Windows Server 2019 Datacenter License:** Giấy phép Datacenter cho phép bạn chạy số lượng máy ảo không giới hạn trên mỗi máy chủ vật lý được cấp phép.
*   Điều này rất lý tưởng cho các mô hình ảo hóa và cụm.
*   **Hiệu suất mạng:** Đảm bảo máy chủ có card mạng Gigabit Ethernet (hoặc cao hơn) để xử lý lưu lượng truy cập từ các Fuzor client.
*   **Phiên bản Fuzor (2017, 2018, 2019, 2025):** Các yêu cầu hệ thống của Fuzor chủ yếu tập trung vào phía client để xử lý đồ họa nặng.
*   Đối với các thành phần server (License và Collaboration), các yêu cầu phần cứng cơ bản thường khá nhất quán qua các phiên bản.
*   Các yêu cầu được liệt kê ở trên là cấu hình chung và thường sẽ đủ cho tất cả các phiên bản Fuzor Server này trên Windows Server 2019 DC.
*   **Collaboration Server:** Fuzor cung cấp tính năng hợp tác Peer-to-Peer (P2P), nơi nhiều bên có thể cùng lúc đắm mình vào cùng một dự án để thảo luận các vấn đề. Host của phiên có thể thay đổi thiết kế và những thay đổi này được khách mời thấy theo thời gian thực. Mặc dù có "Collaboration Server" được đề cập cho "Issue Tracker", nhưng tính năng cộng tác chính dường như diễn ra ở cấp độ P2P, làm giảm tải cho một máy chủ trung tâm.

Tóm lại, Fuzor Server không đòi hỏi tài nguyên khủng về CPU, RAM, và đặc biệt là không cần vGPU. Cấu hình sẽ chủ yếu tập trung vào việc cung cấp một môi trường Windows Server 2019 DC ổn định và đủ tài nguyên để quản lý các tác vụ license và điều phối phiên làm việc một cách hiệu quả.

----

# 2. Cách tính Sizing chi phí triển khai mô hình Fuzor Cluster:

>>>FAQ2. Nếu giả sử tôi vẫn cần tính đến khả năng HA của Fuzor server 2k17-2k25 và sẽ có 2 windows OS 2k19 DC licensed cấu hình Cluster, thì cấu hình kích thước sizing vCPU, vRAM, VD, GPU là bao nhiêu ? để đáp ứng cho 100 kỹ sư dùng: khoảng 200 devices : 100 PC workstation, 100 tablets, 50 iphone os, 100 samsung android mobile app truy xuất sử dụng Fuzor designer và collaborations, hãy miêu tả cả calculator sizing license của Fuzor lúc này sẽ tính như thế nào? chi phí license ?  Cuối cúng, các công cụ backup như Veeam Backup Replicate với mô hình Veeam Agent for windows có backup được cụm Fuzor Cluster windows nói trên ?

---

Để phân tích và cấu hình Fuzor Server cho mô hình cụm (HA cluster) với 2 node Windows Server 2019 Datacenter Edition, đồng thời đáp ứng nhu cầu của 100 kỹ sư sử dụng Fuzor Designer và Collaboration trên nhiều thiết bị, chúng ta cần hiểu rõ vai trò của từng thành phần Fuzor Server và cách chúng tương tác với hạ tầng.

Các phiên bản Fuzor Server 2017, 2018, 2019 và 2025 có thể có những cải tiến về tính năng và hiệu suất, nhưng các yêu cầu cơ bản về phần cứng cho các thành phần server thường không thay đổi đáng kể như với các ứng dụng khách.

### Mô hình triển khai cụm Fuzor Server (HA) trên Windows Server 2019 DC

Fuzor Server không có khả năng clustering nội bộ để đạt được sẵn sàng cao (High Availability). Thay vào đó, HA sẽ được triển khai ở cấp độ hạ tầng ảo hóa và hệ điều hành Windows Server Failover Cluster (WSFC).

1.  **Hạ tầng:** Hai máy chủ vật lý, mỗi máy chạy Windows Server 2019 Datacenter Edition (hoặc Hyper-V Role nếu bạn đang sử dụng Hyper-V).
2.  **Máy ảo:** Tạo hai máy ảo (VM), mỗi VM sẽ được cài đặt Windows Server 2019 Datacenter Edition.
3.  **Fuzor Server Components:**
    *   **Fuzor Network License Server:** Quản lý và phân phối các giấy phép Fuzor cho các client. Đây thường là một dịch vụ nhẹ.
    *   **Fuzor Collaboration Server (Issue Tracker):** Đây là thành phần server cung cấp chức năng theo dõi và quản lý các vấn đề (issues) trong dự án. Nó thường sử dụng một cơ sở dữ liệu (ví dụ: SQLite, hoặc có thể tích hợp với các DB lớn hơn).
    *   **Lưu ý:** Chức năng cộng tác thời gian thực (real-time collaboration) trong Fuzor thường hoạt động theo mô hình ngang hàng (Peer-to-Peer - P2P) giữa các Fuzor client. Điều này có nghĩa là tải nặng về dữ liệu mô hình và tương tác 3D sẽ nằm trên các máy client, không phải trên Fuzor Collaboration Server.

#### Triển khai HA cho Fuzor Server:

*   **License Server:** Có thể cài đặt Fuzor Network License Server trên một trong hai VM và cấu hình nó thành một dịch vụ có sẵn sàng cao trong cụm WSFC (Generic Service). Hoặc đơn giản hơn, nếu các Fuzor client có thể được cấu hình để thử kết nối với nhiều license server, bạn có thể chạy một license server trên mỗi VM và cấu hình client để chuyển đổi giữa chúng. Tuy nhiên, một license server chạy trên cụm WSFC như một nguồn tài nguyên chung là giải pháp phổ biến hơn.
*   **Collaboration Server (Issue Tracker):** Tương tự, nếu Issue Tracker sử dụng một cơ sở dữ liệu, bạn có thể triển khai cơ sở dữ liệu đó trên một SQL Server Cluster hoặc triển khai Issue Tracker trên một VM và làm cho VM đó có sẵn sàng cao trong cụm ảo hóa (Hyper-V/VMware HA).

### Cấu hình kích thước phù hợp (vCPU, vRAM, Virtual Disk, vGPU)

Với 100 kỹ sư sử dụng Fuzor Designer và Collaboration trên tổng số khoảng 200 thiết bị, các yêu cầu về phía Fuzor Server (License & Issue Tracker) không quá cao vì các tác vụ xử lý đồ họa nặng diễn ra trên client.

Dưới đây là cấu hình đề xuất cho **mỗi máy ảo (VM)** chạy Fuzor Server trên Windows Server 2019 Datacenter Edition trong một cụm 2 node:

*   **vCPU:** **4 vCPU**
    *   Mặc dù Fuzor Server không quá nặng, việc có 4 vCPU sẽ đảm bảo Windows Server 2019 và các dịch vụ Fuzor chạy mượt mà, đặc biệt khi có nhiều yêu cầu đồng thời từ 100 người dùng. Nó cũng cung cấp đủ khả năng xử lý cho các tác vụ của hệ điều hành và bất kỳ dịch vụ phụ trợ nào khác.
*   **vRAM:** **16 GB vRAM**
    *   Windows Server 2019 yêu cầu tối thiểu 2GB (GUI) hoặc 512MB (Core). Fuzor License Server và Collaboration Server (Issue Tracker) thường không tiêu thụ nhiều RAM. Tuy nhiên, 16GB sẽ cung cấp một vùng đệm dồi dào, đảm bảo hiệu suất ổn định và cho phép các tác vụ như quản lý cơ sở dữ liệu cho Issue Tracker hoạt động hiệu quả mà không bị thiếu hụt bộ nhớ, đặc biệt khi có hàng trăm lượt truy cập đồng thời.
*   **Virtual Disk (VD):** **200 GB SSD**
    *   **Hệ điều hành:** Khoảng 60-80 GB cho Windows Server 2019.
    *   **Fuzor Server:** Khoảng 10-20 GB cho cài đặt Fuzor License Server và Collaboration Server.
    *   **Dữ liệu:** Phần còn lại để chứa log files, dữ liệu của Issue Tracker (cơ sở dữ liệu), và không gian cho các bản cập nhật.
    *   **Lưu ý quan trọng:** Để triển khai cụm WSFC có tính sẵn sàng cao cho một dịch vụ hoặc vai trò (ví dụ: File Server, SQL Server, hoặc một ứng dụng tùy chỉnh như Fuzor License Server nếu được cấu hình), các ổ đĩa dữ liệu phải nằm trên **bộ nhớ dùng chung (shared storage)** mà cả hai node trong cụm có thể truy cập (ví dụ: SAN, iSCSI, hoặc Shared VHDX trong Hyper-V). Ổ đĩa hệ điều hành có thể là độc lập cho mỗi VM. Sử dụng SSD cho toàn bộ virtual disk để đảm bảo hiệu suất I/O tốt.
*   **vGPU:** **Không bắt buộc**
    *   Fuzor Server (License và Collaboration) không thực hiện bất kỳ công việc kết xuất đồ họa 3D, mô phỏng VR/AR hay 4D/5D nào. Các tác vụ nặng về đồ họa này được thực hiện trên các máy client (PC workstation, tablet, mobile phone). Do đó, việc cấp phát vGPU cho Fuzor Server là không cần thiết và sẽ lãng phí tài nguyên.

### Calculator Sizing License và Chi phí License của Fuzor

Fuzor thường được cấp phép dựa trên số lượng người dùng đồng thời (concurrent users) hoặc người dùng được đặt tên (named users) cho Fuzor Designer.

1.  **Số lượng kỹ sư:** Bạn có 100 kỹ sư.
2.  **Mô hình sử dụng:** 100 kỹ sư này sẽ sử dụng Fuzor Designer và các tính năng cộng tác. Mặc dù có tới 200+ thiết bị, nhưng một người dùng thường chỉ sử dụng một license tại một thời điểm (khi đăng nhập vào Fuzor).
3.  **Loại License:**
    *   **Floating Licenses (Concurrent User Licenses):** Đây là mô hình phổ biến nhất cho các môi trường lớn. Bạn mua một số lượng license nhất định (ví dụ: 50 licenses cho 100 người dùng). Khi một người dùng mở Fuzor Designer, họ sẽ chiếm một license. Khi họ đóng Fuzor, license đó sẽ được trả về pool để người khác sử dụng. Mô hình này giả định rằng không phải tất cả 100 kỹ sư sẽ sử dụng Fuzor cùng một lúc.
    *   **Named User Licenses:** Mỗi kỹ sư được gán một license riêng. Họ có thể sử dụng Fuzor trên bất kỳ thiết bị nào, nhưng chỉ họ mới được dùng license đó. Số lượng license sẽ bằng số lượng người dùng (100 licenses).
    *   **Subscription vs. Perpetual:** Fuzor có thể cung cấp cả hai mô hình này. Subscription là thuê bao hàng năm/tháng, Perpetual là mua license vĩnh viễn với phí bảo trì hàng năm.

**Cách tính License cho 100 kỹ sư:**

*   **Xác định tỷ lệ sử dụng đồng thời (Concurrency Ratio):** Với 100 kỹ sư, không phải lúc nào tất cả họ cũng sử dụng Fuzor. Bạn cần ước tính tỷ lệ sử dụng đồng thời cao nhất. Ví dụ:
    *   Nếu bạn ước tính 70% kỹ sư sẽ sử dụng Fuzor đồng thời (70 concurrent users), bạn sẽ cần 70 license Floating.
    *   Nếu bạn cần đảm bảo tất cả 100 kỹ sư đều có thể sử dụng Fuzor bất cứ lúc nào, bạn sẽ cần 100 license (có thể là Floating licenses với 100 slots, hoặc 100 Named User licenses).
    *   Tỷ lệ 1:1.5 hoặc 1:2 (ví dụ: 100 người dùng cần 50-70 license) thường được áp dụng cho các phần mềm thiết kế chuyên dụng. Với 100 kỹ sư, bạn có thể cân nhắc từ **60 đến 80 Floating Licenses** để đảm bảo hầu hết các trường hợp sử dụng.

**Chi phí License:**

*   **Ước tính (chỉ mang tính tham khảo):** Giá Fuzor có thể thay đổi tùy thuộc vào phiên bản, tính năng đi kèm (ví dụ: VR/AR, 4D/5D, Issue Tracker), số lượng license và chính sách của nhà cung cấp.
    *   Một license Fuzor Designer có thể có giá từ **vài nghìn USD đến trên 10.000 USD/license/năm** cho mô hình subscription hoặc chi phí ban đầu cao hơn nhiều cho perpetual license cộng với phí bảo trì hàng năm.
    *   Với giả định bạn cần 60-80 Floating Licenses, tổng chi phí ban đầu có thể lên tới **hàng trăm nghìn USD** (ví dụ: 60 licenses x $5,000/license = $300,000/năm cho subscription, hoặc chi phí ban đầu cao hơn cho perpetual).
    *   **Lời khuyên:** Để có báo giá chính xác nhất, bạn cần liên hệ trực tiếp với sales của Fuzor hoặc các đại lý phân phối chính thức của họ và cung cấp thông tin chi tiết về nhu cầu của bạn (số lượng người dùng, tính năng yêu cầu, mô hình license mong muốn).
*   **Fuzor Network License Server License:** Thường đi kèm miễn phí hoặc với số lượng license bạn mua. Nó không yêu cầu license riêng.
*   **Fuzor Collaboration Server (Issue Tracker):** Có thể là một phần của gói Fuzor Designer hoặc là một module bổ sung với chi phí riêng.

### Công cụ Backup: Veeam Backup & Replication với Veeam Agent for Windows

Có, **Veeam Backup & Replication với Veeam Agent for Windows hoàn toàn có thể sao lưu cụm Fuzor Server Windows nói trên của bạn.**

Cách tiếp cận sẽ phụ thuộc vào việc bạn triển khai Fuzor Server dưới dạng máy ảo hay máy vật lý và cách cụm được cấu hình:

1.  **Nếu Fuzor Server được triển khai trên các Máy ảo (VMs) trong môi trường ảo hóa (Hyper-V hoặc VMware):**
    *   **Veeam Backup & Replication** là giải pháp lý tưởng. Nó sẽ tích hợp trực tiếp với Hypervisor (Hyper-V hoặc VMware vSphere) của bạn.
    *   Veeam sẽ sao lưu toàn bộ máy ảo, bao gồm hệ điều hành (Windows Server 2019 DC), các ứng dụng Fuzor Server và dữ liệu trên đó.
    *   Veeam sử dụng tính năng **Application-Aware Processing** (sử dụng Microsoft VSS - Volume Shadow Copy Service) để đảm bảo rằng các ứng dụng và cơ sở dữ liệu bên trong VM (ví dụ: SQL Server nếu Issue Tracker sử dụng) được sao lưu ở trạng thái nhất quán, có thể khôi phục được. Điều này cực kỳ quan trọng đối với dữ liệu động.
    *   Veeam cũng hỗ trợ sao lưu các VM trong cụm Windows Server Failover Cluster, đảm bảo rằng ngay cả khi một VM chuyển đổi sang node khác, quá trình sao lưu vẫn diễn ra bình thường.

2.  **Nếu Fuzor Server được triển khai trên Máy vật lý hoặc cần sao lưu cấp độ hệ điều hành/ứng dụng bên trong VM:**
    *   **Veeam Agent for Windows** (trước đây là Veeam Endpoint Backup) có thể được cài đặt trực tiếp lên từng máy chủ Windows Server 2019 DC trong cụm.
    *   Veeam Agent có thể sao lưu toàn bộ máy tính, các ổ đĩa riêng lẻ, hoặc các thư mục cụ thể.
    *   Nó cũng hỗ trợ **Application-Aware Processing** để đảm bảo tính nhất quán của dữ liệu ứng dụng.
    *   Đối với các máy chủ trong cụm WSFC, Veeam Agent for Windows có khả năng nhận biết cụm, có thể sao lưu các tài nguyên dùng chung của cụm (Cluster Shared Volumes - CSV) và đảm bảo các ứng dụng cụm được sao lưu chính xác.

**Kết luận về Backup:**

*   **Khuyến nghị mạnh mẽ nhất:** Nếu Fuzor Server của bạn là VM, hãy sử dụng **Veeam Backup & Replication** để sao lưu ở cấp độ ảo hóa. Đây là cách hiệu quả và mạnh mẽ nhất để bảo vệ môi trường ảo.
*   **Tùy chọn bổ sung:** Nếu có yêu cầu sao lưu cụ thể hơn ở cấp độ ứng dụng bên trong VM hoặc nếu bạn có một số thành phần Fuzor Server chạy trên máy vật lý, **Veeam Agent for Windows** là một giải pháp bổ sung hoặc thay thế phù hợp.

Veeam là một giải pháp hàng đầu trong việc sao lưu và phục hồi dữ liệu cho cả môi trường vật lý và ảo, và nó hoàn toàn tương thích để bảo vệ các cụm Windows Server và các ứng dụng chạy trên đó như Fuzor Server.

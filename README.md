# StoryWeaver
Dự án StoryWeaver AI: Tài Liệu Kiến Trúc & Kỹ Thuật

Version: 1.0

Ngày cập nhật: 24-09-2025

Mô tả: Tài liệu này là nguồn thông tin trung tâm (single source of truth) về kiến trúc hệ thống, công nghệ sử dụng và quy trình phát triển cho dự án StoryWeaver AI.

1. Giới Thiệu Dự Án (Project Overview)

StoryWeaver AI là một nền tảng SaaS (Software as a Service) đột phá, sử dụng Trí tuệ nhân tạo để tự động hóa quy trình sản xuất video. Mục tiêu của chúng tôi là biến những ý tưởng, kịch bản và các media (hình ảnh, video) rời rạc của người dùng thành một video câu chuyện hoàn chỉnh, chuyên nghiệp và có cảm xúc chỉ trong vài phút, thay vì hàng giờ hoặc hàng ngày.

Vấn đề giải quyết: Quy trình sản xuất video truyền thống tốn kém, tốn thời gian và đòi hỏi kỹ năng chuyên môn cao.

Giải pháp của chúng tôi: Một nền tảng thông minh, tự động hóa các bước từ phân tích kịch bản, lựa chọn media, dựng phim, lồng tiếng, đến xuất bản video.

2. Kiến Trúc Hệ Thống Tổng Thể (Overall System Architecture)

Chúng tôi áp dụng kiến trúc hướng dịch vụ (Service-Oriented Architecture), tách biệt rõ ràng các khối chức năng để đảm bảo tính linh hoạt, khả năng mở rộng và dễ dàng bảo trì.

2.1. Sơ Đồ Kiến Trúc
code
Mermaid
download
content_copy
expand_less
graph TD
    subgraph "Người Dùng"
        User[<fa:fa-user> Người Dùng]
    end

    subgraph "Nền Tảng Cloud"
        Frontend( <fa:fa-window-maximize> Frontend Web App <br> React/Vue.js )
        Backend( <fa:fa-server> Backend Services <br> API Gateway, Business Logic <br> Python/FastAPI )
        
        subgraph "AI Core Services (Microservices)"
            CV_Service[ <fa:fa-camera> Computer Vision Service <br> Gán nhãn, nhận diện đối tượng ]
            NLP_Service[ <fa:fa-file-alt> NLP Service <br> Phân tích kịch bản ]
            Match_Service[ <fa:fa-link> Smart Matching Service <br> Ghép nối media & kịch bản ]
            TTS_Service[ <fa:fa-microphone> Text-to-Speech Service <br> Tạo giọng đọc ]
        end

        subgraph "Hệ Thống Xử Lý Nền"
            Queue( <fa:fa-exchange-alt> Message Queue <br> RabbitMQ/SQS )
            VideoWorker( <fa:fa-cogs> Video Processing Workers <br> FFmpeg, MoviePy )
        end

        subgraph "Lưu Trữ"
            DB[ <fa:fa-database> Cơ sở dữ liệu <br> PostgreSQL ]
            Storage[ <fa:fa-hdd> Lưu trữ file <br> AWS S3 / MinIO ]
        end
    end

    User --> Frontend
    Frontend --> Backend

    Backend --> DB
    Backend --> Storage
    Backend --> Queue
    
    Queue --> CV_Service
    Queue --> NLP_Service
    Queue --> Match_Service
    Queue --> TTS_Service
    Queue --> VideoWorker
    
    CV_Service --> Backend
    NLP_Service --> Backend
    Match_Service --> Backend
    TTS_Service --> Backend
    VideoWorker --> Storage
    VideoWorker --> Backend
2.2. Mô Tả Các Thành Phần

Frontend Web App:

Trách nhiệm: Là giao diện người dùng duy nhất. Xây dựng bằng React/Vue.js.

Nhiệm vụ: Cung cấp trải nghiệm mượt mà cho việc tải media, soạn thảo kịch bản, quản lý dự án và xem trước/tải video thành phẩm.

Giao tiếp với hệ thống hoàn toàn qua RESTful API do Backend cung cấp.

Backend Services (Python/FastAPI):

Trách nhiệm: Là bộ não điều phối toàn bộ hệ thống.

Nhiệm vụ:

Cung cấp RESTful API cho Frontend.

Xử lý logic nghiệp vụ: quản lý người dùng, dự án, thanh toán.

Giao tiếp với CSDL để lưu trữ metadata.

Đẩy các tác vụ nặng (phân tích AI, render video) vào Message Queue để xử lý bất đồng bộ.

Nhận kết quả từ các worker và cập nhật trạng thái cho người dùng (qua WebSocket hoặc thông báo).

AI Core Services (Microservices):

Trách nhiệm: Thực hiện các tác vụ Trí tuệ nhân tạo chuyên biệt. Mỗi service là một API độc lập.

Các services chính: Computer Vision, NLP, Smart Matching, Text-to-Speech.

Nhận tác vụ từ Message Queue và xử lý.

Hệ Thống Xử Lý Nền (Background Processing):

Message Queue (RabbitMQ/SQS): Đóng vai trò hàng đợi, giúp hệ thống chịu tải tốt và xử lý các tác vụ dài hơi mà không làm block request của người dùng.

Video Processing Workers: Các service chuyên biệt nhận nhiệm vụ từ Message Queue để thực hiện các thao tác xử lý video nặng như cắt, ghép, chèn âm thanh, và render video cuối cùng.

Lưu Trữ (Storage):

Cơ sở dữ liệu (PostgreSQL): Lưu trữ dữ liệu có cấu trúc như thông tin người dùng, metadata dự án, kịch bản, kết quả phân tích AI...

Lưu trữ file (AWS S3/MinIO): Lưu trữ các file media dung lượng lớn (ảnh, video gốc, audio, video đã render).

3. Luồng Hoạt Động Chính (Core Workflow)
Luồng tạo video từ kịch bản:

(Người dùng & Frontend): Người dùng tạo một dự án mới, tải lên các file media (ảnh, video) và dán kịch bản văn bản vào trình soạn thảo.

(Frontend -> Backend): Frontend gọi API POST /api/v1/projects đến Backend, gửi kèm kịch bản và danh sách các file media đã được tải lên S3.

(Backend):

Backend nhận request, xác thực người dùng.

Lưu thông tin dự án (metadata) vào CSDL PostgreSQL với trạng thái PROCESSING.

Đẩy một loạt các "công việc" (jobs) vào Message Queue. Ví dụ:

Job 1: { project_id: 123, task: 'analyze_script' }

Job 2: { project_id: 123, media_id: 'abc.jpg', task: 'analyze_image' }

Job 3: { project_id: 123, media_id: 'def.mp4', task: 'analyze_video' }

Trả về cho Frontend một response ngay lập tức với project_id và trạng thái PROCESSING, để người dùng không phải chờ đợi.

(AI Services):

Các worker của NLP Service và Computer Vision Service lắng nghe Queue, nhận các job tương ứng.

Chúng xử lý, phân tích và lưu kết quả (ví dụ: các tags của ảnh, các keywords của kịch bản) vào CSDL.

(Backend): Sau khi các job phân tích hoàn thành, Backend đẩy một job mới vào Queue:

Job 4: { project_id: 123, task: 'generate_timeline' }

(Smart Matching Service): Worker của dịch vụ này nhận job, đọc kết quả phân tích từ CSDL và sử dụng thuật toán để tạo ra một "dòng thời gian" (timeline) ảo, quyết định clip/ảnh nào sẽ đi với câu kịch bản nào, dài bao nhiêu giây. Kết quả timeline này được lưu vào CSDL.

(Backend): Sau khi có timeline, Backend đẩy job cuối cùng:

Job 5: { project_id: 123, task: 'render_video' }

(Video Processing Workers): Worker xử lý video nhận job này. Nó đọc timeline từ CSDL, tải các file media gốc từ S3, sử dụng FFmpeg/MoviePy để cắt, ghép, thêm nhạc/giọng đọc, và render thành file video cuối cùng.

(Worker -> S3 & Backend): Video thành phẩm được tải lên S3. Worker cập nhật trạng thái dự án trong CSDL thành COMPLETED và thông báo cho Backend.

(Backend -> Frontend): Backend gửi một thông báo (ví dụ qua WebSocket) cho Frontend rằng video đã sẵn sàng. Người dùng nhận được thông báo và có thể xem hoặc tải video.

4. Công Nghệ Sử Dụng (Technology Stack)

Frontend: Vue.js (hoặc React), TypeScript, Tailwind CSS

Backend: Python 3.9+, FastAPI, SQLAlchemy, Pydantic

AI/ML: PyTorch, Transformers, OpenCV, spaCy

Video Processing: FFmpeg, MoviePy

Cơ sở dữ liệu: PostgreSQL

Lưu trữ file: AWS S3 (Production), MinIO (Development)

Message Queue: RabbitMQ (hoặc AWS SQS)

Hạ tầng & DevOps: Docker, Kubernetes (K8s), AWS (EC2, S3, RDS), CI/CD (GitHub Actions)

5. Quy Trình Phát Triển (Development Workflow)

Quản lý mã nguồn: Git.

Quy trình Git: Gitflow (sử dụng các nhánh main, develop, feature/*, hotfix/*). Mọi thay đổi phải được đưa vào qua Pull Request và cần ít nhất 1 review.

Quản lý công việc: Jira / Trello (Sử dụng board Kanban).

Giao tiếp: Slack / Discord.

Tài liệu API: Swagger UI (tự động tạo bởi FastAPI).

6. Hướng Dẫn Cài Đặt Môi Trường (Setup Guide)

Mỗi service (Backend, Frontend, AI Services) sẽ có một file README.md chi tiết riêng. Dưới đây là các bước chung:

Clone repository: git clone [URL]

Cài đặt Docker & Docker Compose.

Tạo file môi trường: Sao chép file .env.example thành .env và điền các thông tin cấu hình cần thiết (database URL, S3 credentials, etc.).

Chạy dự án: Chạy lệnh docker-compose up --build từ thư mục gốc của dự án.

Truy cập:

Frontend: http://localhost:3000

Backend API Docs: http://localhost:8000/docs

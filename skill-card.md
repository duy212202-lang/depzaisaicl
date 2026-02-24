# Skill Card: UGC Comment Insights Miner

| Mục | Nội dung |
|---|---|
| **Tên Skill** | UGC Comment Insights Miner |
| **Việc gì được automate** | Thu thập, lọc, và phân loại community insights từ comment threads (Reddit, X, YouTube, forums) thành các content sections sẵn sàng dùng cho blog — bao gồm counter-arguments, FAQ từ real users, personal experiences, và viral pattern analysis |
| **TRƯỚC: làm tay** | Mỗi bài blog mất 1–2 giờ lướt Reddit/X/forum, đọc hàng trăm comments, cherry-pick thủ công, tự viết lại. Hầu hết writer bỏ qua bước này vì tốn thời gian → bài viết ra thiếu UGC depth |
| **SAU: có skill** | Paste 1–3 URL → 1–2 phút → output markdown report hoàn chỉnh với 5 sections (Executive Summary, Counter-Arguments, FAQ, Experiences, Viral Analysis) + optional Blog Cross-Reference. Copy-paste thẳng vào blog, chỉnh sửa < 5 phút |
| **Tool/AI đã dùng** | Claude (SKILL.md prompt engineering), web_fetch (URL fetching), Claude Projects |
| **Limitation** | X/Twitter có thể block scraping (cần backup Reddit links); JS-rendered comments không fetch được; giới hạn top 100 comments/URL; chưa có scheduled monitoring |
| **Roadmap mở rộng** | Phase 2: Multi-link batch (5–10 URLs), template variants theo blog type, brand voice calibration, DOCX output. Phase 3: Browser extension "X Learning Companion" — smart bookmarks, profile tracker, daily digest |

---
title : "Thiáº¿t láº­p TÃ i Khoáº£n AWS"
date :  "2026-10-03" 
weight : 1 
chapter : false
---

# Táº¡o tÃ i khoáº£n AWS Ä‘áº§u tiÃªn

#### Tá»•ng quan
Trong bÃ i lab Ä‘áº§u tiÃªn nÃ y, báº¡n sáº½ táº¡o má»›i **tÃ i khoáº£n AWS** Ä‘áº§u tiÃªn cá»§a mÃ¬nh, táº¡o **MFA** (Multi-factor Authentication) Ä‘á»ƒ gia tÄƒng báº£o máº­t tÃ i khoáº£n cá»§a báº¡n. BÆ°á»›c tiáº¿p theo báº¡n sáº½ táº¡o **Admin Group**, **Admin User** Ä‘á»ƒ quáº£n lÃ½ quyá»n truy cáº­p vÃ o cÃ¡c tÃ i nguyÃªn trong tÃ i khoáº£n cá»§a mÃ¬nh thay vÃ¬ sá»­ dá»¥ng user root.\
Cuá»‘i cÃ¹ng, náº¿u quÃ¡ trÃ¬nh xÃ¡c thá»±c tÃ i khoáº£n cá»§a báº¡n cÃ³ váº¥n Ä‘á», báº¡n sáº½ Ä‘Æ°á»£c hÆ°á»›ng dáº«n há»— trá»£ xÃ¡c thá»±c tÃ i khoáº£n vá»›i **AWS Support**.

#### TÃ i khoáº£n AWS (AWS Account)
**TÃ i khoáº£n AWS** lÃ  phÆ°Æ¡ng tiá»‡n Ä‘á»ƒ báº¡n cÃ³ thá»ƒ truy cáº­p vÃ  sá»­ dá»¥ng nhá»¯ng tÃ i nguyÃªn vÃ  dá»‹ch vá»¥ cá»§a AWS. Theo máº·c Ä‘á»‹nh, má»—i tÃ i khoáº£n AWS sáº½ cÃ³ má»™t *root user*. *Root user* cÃ³ toÃ n quyá»n vá»›i tÃ i khoáº£n AWS cá»§a báº¡n, vÃ  quyá»n háº¡n cá»§a root user khÃ´ng thá»ƒ bá»‹ giá»›i háº¡n. Náº¿u báº¡n má»›i sá»­ dá»¥ng tÃ i khoáº£n AWS láº§n Ä‘áº§u tiÃªn, báº¡n sáº½ truy cáº­p vÃ o tÃ i khoáº£n dÆ°á»›i danh nghÄ©a cá»§a *root user*.

![Create Account](/images/1/0001.png?featherlight=false&width=90pc)

{{% notice note %}}
ChÃ­nh vÃ¬ quyá»n háº¡n cá»§a **root user** khÃ´ng thá»ƒ bá»‹ giá»›i háº¡n, AWS khuyÃªn báº¡n khÃ´ng nÃªn sá»­ dá»¥ng trá»±c tiáº¿p *root user* cho báº¥t ká»³ cÃ´ng tÃ¡c nÃ o. Thay vÃ o Ä‘Ã³, báº¡n nÃªn táº¡o ra má»™t *IAM User* vÃ  trao quyá»n quáº£n trá»‹ cho *IAM User* Ä‘Ã³ Ä‘á»ƒ dá»… dÃ ng quáº£n lÃ½ vÃ  giáº£m thiá»ƒu rá»§i ro.
{{% /notice %}}

#### MFA (Multi-factor Authentication)
**MFA** lÃ  má»™t tÃ­nh nÄƒng Ä‘Æ°á»£c sá»­ dá»¥ng Ä‘á»ƒ gia tÄƒng báº£o máº­t cá»§a tÃ i khoáº£n AWS. Náº¿u MFA Ä‘Æ°á»£c kÃ­ch hoáº¡t, báº¡n sáº½ pháº£i nháº­p mÃ£ OTP (One-time Password) má»—i láº§n báº¡n Ä‘Äƒng nháº­p vÃ o tÃ i khoáº£n AWS.

#### IAM Group 
**IAM Group**  lÃ  má»™t cÃ´ng cá»¥ quáº£n lÃ½ ngÆ°á»i dÃ¹ng (*IAM User*) cá»§a AWS. Má»™t IAM Group cÃ³ thá»ƒ chá»©a nhiá»u IAM User. CÃ¡c IAM User á»Ÿ trong má»™t IAM Group Ä‘á»u hÆ°á»Ÿng chung quyá»n háº¡n mÃ  IAM Group Ä‘Ã³ Ä‘Æ°á»£c gÃ¡n cho.

#### IAM User
**IAM User** lÃ  má»™t Ä‘Æ¡n vá»‹ ngÆ°á»i dÃ¹ng cá»§a AWS. Khi báº¡n Ä‘Äƒng nháº­p vÃ o AWS, báº¡n sáº½ pháº£i Ä‘Äƒng nháº­p dÆ°á»›i danh nghÄ©a cá»§a má»™t IAM User. Náº¿u báº¡n má»›i Ä‘Äƒng nháº­p vÃ o AWS láº§n Ä‘áº§u tiÃªn, báº¡n sáº½ Ä‘Äƒng nháº­p dÆ°á»›i danh nghÄ©a cá»§a *root user* (táº¡m dá»‹ch lÃ  ngÆ°á»i dÃ¹ng gá»‘c). NgoÃ i *root user* ra, báº¡n cÃ³ thá»ƒ táº¡o ra nhiá»u IAM User khÃ¡c Ä‘á»ƒ cho phÃ©p ngÆ°á»i khÃ¡c truy cáº­p **dÃ i háº¡n** vÃ o tÃ i nguyÃªn AWS trong tÃ i khoáº£n AWS cá»§a báº¡n.


#### AWS Support
**AWS Support** lÃ  má»™t Ä‘Æ¡n vá»‹ cung cáº¥p cÃ¡c dá»‹ch vá»¥ há»— trá»£ khÃ¡ch hÃ ng cá»§a AWS.


#### Ná»™i dung chÃ­nh

1. [Táº¡o tÃ i khoáº£n AWS](1-create-new-aws-account/)
2. [Thiáº¿t láº­p MFA cho tÃ i khoáº£n AWS (Root)](2-mfa-setup-for-aws-user-(root)/)
3. [TÃ i khoáº£n vÃ  NhÃ³m Admin](3-create-admin-user-and-group/)
4. [Há»— trá»£ XÃ¡c thá»±c TÃ i khoáº£n](4-verify-new-account/)

---
title : "Táº¡o Admin Group vÃ  Admin User"
date :  "2026-10-03" 
weight : 3
chapter : false
pre : " <b> 3. </b> "
---


## Táº¡o Admin Group

1. **ÄÄƒng nháº­p vÃ o Báº£ng Ä‘iá»u khiá»ƒn** á»Ÿ trang [AWS Web Service page](https://aws.amazon.com/)

2. Nháº¥n vÃ o tÃªn tÃ i khoáº£n á»Ÿ gÃ³c trÃªn bÃªn pháº£i vÃ  chá»n **My Security Credentials**

   ![AWS IAM](/images/01/0001.png?featherlight=false&width=90pc)

   > **LÆ°u Ã½:** Trong trÆ°á»ng há»£p khÃ´ng tháº¥y menu **My Security Credentials**, báº¡n cÃ³ thá»ƒ click vÃ o biá»ƒu tÆ°á»£ng tÃ¬m kiáº¿m vÃ  Ä‘iá»n **IAM**. Sau Ä‘Ã³ click vÃ o dá»‹ch vá»¥ IAM Ä‘á»ƒ truy cáº­p vÃ o giao diá»‡n quáº£n lÃ½ dá»‹ch vá»¥ IAM.

   ![AWS IAM](/images/01/0002.png?featherlight=false&width=90pc)

3. á»ž thanh bÃªn trÃ¡i, chá»n **User Groups** sau Ä‘Ã³ chá»n **Create Group**

   ![AWS IAM](/images/01/0003.png?featherlight=false&width=90pc)

4. DÆ°á»›i má»¥c **Name the group**, nháº­p tÃªn Group (VÃ­ dá»¥: *AdminGroup*) vÃ  cuá»™n chuá»™t xuá»‘ng dÆ°á»›i

   ![AWS IAM](/images/01/0004.png?featherlight=false&width=90pc)

5. á»ž pháº§n **Attach permissions policies**, gÃµ **AdministratorAccesss** vÃ o thanh tÃ¬m kiáº¿m vÃ  nháº¥n chá»n nÃ³. Cuá»‘i cÃ¹ng, chá»n **Create Group**.

   ![AWS IAM](/images/01/0005.png?featherlight=false&width=90pc)

6. HoÃ n thÃ nh táº¡o admin group.

   ![AWS IAM](/images/01/0006.png?featherlight=false&width=90pc)

## Táº¡o Admin User

1. á»ž thanh bÃªn trÃ¡i, chá»n **Users** sau Ä‘Ã³ chá»n **Add User**

   ![AWS IAM](/images/02/0001.png?featherlight=false&width=90pc)

2. Nháº­p tÃªn User (VÃ­ dá»¥: *AdminUser*).
    + Click **AWS Management Console access**.
    + Click **Programmatic Access**.
    + Click **Custom password** rá»“i gÃµ má»™t password tÃ¹y Ã½ cá»§a báº¡n (lÆ°u Ã½: báº¡n pháº£i ghi nhá»› máº­t kháº©u nÃ y cho nhá»¯ng láº§n Ä‘Äƒng nháº­p trong tÆ°Æ¡ng lai).
    + Bá» chá»n má»¥c **User must create a new password...**.
    + Click **Next:Permissions**.

   ![AWS IAM](/images/02/0002.png?featherlight=false&width=90pc)

   > **LÆ°u Ã½:** Báº±ng cÃ¡ch chá»n **AWS Management Console access**, báº¡n vá»«a cho phÃ©p IAM User Ä‘Æ°á»£c truy cáº­p vÃ o AWS thÃ´ng qua báº£ng Ä‘iá»u khiá»ƒn AWS trÃªn web. Viá»‡c bá» má»¥c **User must create a new password...** cho phÃ©p ngÆ°á»i dÃ¹ng khi láº§n Ä‘áº§u Ä‘Äƒng nháº­p vÃ o IAM User Ä‘Ã³ khÃ´ng cáº§n pháº£i táº¡o máº­t kháº©u má»›i.

3. Click tab **Add user to group** vÃ  click **AdminGroup** mÃ  chÃºng ta táº¡o trÆ°á»›c Ä‘Ã³.

   ![AWS IAM](/images/02/0003.png?featherlight=false&width=90pc)

4. Click **Next:Tags**
    - Tags (tháº») lÃ  má»™t tÃ¹y chá»n khÃ´ng báº¯t buá»™c Ä‘á»ƒ tá»• chá»©c, theo dÃµi, hoáº·c Ä‘iá»u khiá»ƒn truy cáº­p cá»§a user, tháº¿ nÃªn báº¡n cÃ³ thá»ƒ thÃªm tags hoáº·c khÃ´ng.

5. Click **Next:Review**.

   ![AWS IAM](/images/02/0004.png?featherlight=false&width=90pc)

6. Kiá»ƒm tra thÃ´ng tin vÃ  chá»n **Create user**

   ![AWS IAM](/images/02/0005.png?featherlight=false&width=90pc)

7. HoÃ n thÃ nh táº¡o user. CÃ³ thá»ƒ download.csv Ä‘á»ƒ lÆ°u trá»¯ Access key.

   ![AWS IAM](/images/02/0006.png?featherlight=false&width=90pc)

8. Táº¡o admin user thÃ nh cÃ´ng.

   ![AWS IAM](/images/02/0007.png?featherlight=false&width=90pc)

9. Kiá»ƒm tra thÃ´ng tin chi tiáº¿t user.

   ![AWS IAM](/images/02/0008.png?featherlight=false&width=90pc)

   > **LÆ°u Ã½:** Sau khi táº¡o user, báº¡n sáº½ tháº¥y hiá»‡n lÃªn há»™p thoáº¡i download thÃ´ng tin access key vÃ  secret key. ÄÃ¢y lÃ  thÃ´ng tin dÃ¹ng Ä‘á»ƒ thá»±c hiá»‡n **Programmatic access** tá»›i cÃ¡c tÃ i nguyÃªn cá»§a AWS thÃ´ng qua **AWS CLI** vÃ  **AWS SDK**. Táº¡m thá»i chÃºng ta sáº½ chÆ°a sá»­ dá»¥ng tá»›i.

# ÄÄƒng nháº­p vÃ o AdminUser

1. Trá»Ÿ vá» dá»‹ch vá»¥ IAM, vÃ  chá»n **Users** á»Ÿ thanh bÃªn trÃ¡i.
2. Nháº¥n vÃ o tÃªn cá»§a IAM User mÃ  báº¡n vá»«a chá»n.
3. Trong má»¥c **Summary**, chá»n tab **Security credentials**. NhÃ¬n vÃ o dÃ²ng **Summary: Console sign-in link** vÃ  copy Ä‘Æ°á»ng link bÃªn cáº¡nh nÃ³. ÄÃ¢y lÃ  Ä‘Æ°á»ng link báº¡n dÃ¹ng Ä‘á»ƒ Ä‘Äƒng nháº­p vÃ o IAM User.

   ![AWS IAM](/images/03/0001.png?featherlight=false&width=90pc)

4. Má»Ÿ má»™t tab áº©n danh cá»§a trÃ¬nh duyá»‡t báº¡n Ä‘ang sá»­ dá»¥ng vÃ  paste Ä‘Æ°á»ng link áº¥y vÃ o thanh tÃ¬m kiáº¿m.

   ![AWS IAM](/images/03/0002.png?featherlight=false&width=90pc)

   > **LÆ°u Ã½:** Viá»‡c Ä‘Äƒng nháº­p báº±ng tab áº©n danh cho phÃ©p báº¡n Ä‘Äƒng nháº­p vÃ o AWS báº±ng IAM User mÃ  khÃ´ng cáº§n pháº£i log out khá»i root user trong tab chÃ­nh.

5. Nháº­p Ä‘Ãºng tÃªn IAM User vÃ  password mÃ  báº¡n Ä‘Ã£ nháº­p á»Ÿ pháº§n **táº¡o IAM User** á»Ÿ trÃªn. Nháº¥n **sign in**.

   ![AWS IAM](/images/03/0003.png?featherlight=false&width=90pc)

6. ChÃºc má»«ng, báº¡n Ä‘Ã£ truy cáº­p thÃ nh cÃ´ng vÃ o tÃ i khoáº£n cá»§a báº¡n dÆ°á»›i danh nghÄ©a cá»§a má»™t IAM User **AdminUser**.

   ![AWS IAM](/images/03/0004.png?featherlight=false&width=90pc)

7. BÆ°á»›c tiáº¿p theo, chÃºng ta sáº½ chuyá»ƒn sang sá»­ dá»¥ng IAM Role Ä‘á»ƒ nÃ¢ng cao tÃ­nh báº£o máº­t hÆ¡n cho account cá»§a báº¡n nhÃ©.


#### Tham kháº£o

# IAM User vÃ  ÄÄƒng Nháº­p Trong AWS

Má»™t IAM user lÃ  má»™t Ä‘á»‹nh danh Ä‘Æ°á»£c táº¡o trong má»™t tÃ i khoáº£n AWS, cÃ³ quyá»n Ä‘Æ°á»£c tÆ°Æ¡ng tÃ¡c vá»›i cÃ¡c tÃ i nguyÃªn AWS. NgÆ°á»i dÃ¹ng IAM Ä‘Äƒng nháº­p báº±ng cÃ¡ch sá»­ dá»¥ng ID tÃ i khoáº£n hoáº·c tÃªn bÃ­ danh, tÃªn ngÆ°á»i dÃ¹ng cá»§a há» vÃ  máº­t kháº©u. TÃªn ngÆ°á»i dÃ¹ng IAM Ä‘Æ°á»£c cáº¥u hÃ¬nh bá»Ÿi quáº£n trá»‹ viÃªn cá»§a báº¡n. TÃªn ngÆ°á»i dÃ¹ng IAM cÃ³ thá»ƒ lÃ  tÃªn thÃ¢n thiá»‡n nhÆ° "Zhang", hoáº·c lÃ  Ä‘á»‹a chá»‰ email nhÆ° "zhang@example.com". TÃªn ngÆ°á»i dÃ¹ng IAM khÃ´ng thá»ƒ chá»©a khoáº£ng tráº¯ng, nhÆ°ng cÃ³ thá»ƒ chá»©a chá»¯ cÃ¡i in hoa vÃ  thÆ°á»ng, sá»‘, vÃ  cÃ¡c kÃ½ tá»± + = , . @ _ -.

**Máº¹o**
Náº¿u ngÆ°á»i dÃ¹ng IAM cá»§a báº¡n Ä‘Ã£ báº­t xÃ¡c thá»±c Ä‘a yáº¿u tá»‘ (MFA), báº¡n pháº£i cÃ³ quyá»n truy cáº­p vÃ o thiáº¿t bá»‹ xÃ¡c thá»±c. Äá»ƒ biáº¿t thÃªm chi tiáº¿t, xem [Sá»­ dá»¥ng thiáº¿t bá»‹ MFA vá»›i trang Ä‘Äƒng nháº­p IAM cá»§a báº¡n](link_to_MFA_documentation).

**Äá»ƒ Ä‘Äƒng nháº­p vá»›i tÆ° cÃ¡ch ngÆ°á»i dÃ¹ng IAM**
1. Má»Ÿ Báº£ng Ä‘iá»u khiá»ƒn Quáº£n lÃ½ AWS táº¡i https://console.aws.amazon.com/.

2. Trang Ä‘Äƒng nháº­p chÃ­nh hiá»ƒn thá»‹. Chá»n "IAM user", nháº­p ID tÃ i khoáº£n (12 chá»¯ sá»‘) hoáº·c bÃ­ danh vÃ  chá»n "Next".

**Ghi chÃº**
Báº¡n cÃ³ thá»ƒ khÃ´ng cáº§n pháº£i nháº­p ID tÃ i khoáº£n hoáº·c bÃ­ danh náº¿u báº¡n Ä‘Ã£ Ä‘Äƒng nháº­p trÆ°á»›c Ä‘Ã³ vá»›i tÃ i khoáº£n IAM báº±ng trÃ¬nh duyá»‡t hiá»‡n táº¡i hoáº·c náº¿u báº¡n Ä‘ang sá»­ dá»¥ng URL Ä‘Äƒng nháº­p tÃ i khoáº£n cá»§a báº¡n.

3. Nháº­p tÃªn ngÆ°á»i dÃ¹ng IAM vÃ  máº­t kháº©u cá»§a báº¡n, sau Ä‘Ã³ chá»n "Sign in".

4. Náº¿u MFA Ä‘Æ°á»£c báº­t cho ngÆ°á»i dÃ¹ng IAM cá»§a báº¡n, sau Ä‘Ã³ báº¡n sáº½ Ä‘Æ°á»£c xÃ¡c thá»±c sá»­ dá»¥ng nÃ³.

Sau khi xÃ¡c thá»±c, Báº£ng Ä‘iá»u khiá»ƒn Quáº£n lÃ½ AWS sáº½ má»Ÿ trang chá»§ Báº£ng Ä‘iá»u khiá»ƒn. 


### Táº¡o khÃ³a truy cáº­p (access key) cho tÃ i khoáº£n root user trÃªn AWS

#### Quyá»n tá»‘i thiá»ƒu cáº§n thiáº¿t

Äá»ƒ thá»±c hiá»‡n cÃ¡c bÆ°á»›c sau, báº¡n cáº§n cÃ³ Ã­t nháº¥t cÃ¡c quyá»n IAM (Identity and Access Management) sau Ä‘Ã¢y:

- Báº¡n pháº£i Ä‘Äƒng nháº­p vá»›i tÆ° cÃ¡ch lÃ  tÃ i khoáº£n root user cá»§a AWS, Ä‘iá»u nÃ y khÃ´ng yÃªu cáº§u báº¥t ká»³ quyá»n IAM bá»• sung nÃ o khÃ¡c. Báº¡n khÃ´ng thá»ƒ thá»±c hiá»‡n cÃ¡c bÆ°á»›c nÃ y vá»›i tÆ° cÃ¡ch lÃ  ngÆ°á»i dÃ¹ng IAM hoáº·c vai trÃ² (role).

- Sá»­ dá»¥ng Ä‘á»‹a chá»‰ email vÃ  máº­t kháº©u cá»§a tÃ i khoáº£n AWS Ä‘á»ƒ Ä‘Äƒng nháº­p vÃ o Báº£ng Ä‘iá»u khiá»ƒn quáº£n lÃ½ AWS (AWS Management Console) vá»›i tÆ° cÃ¡ch lÃ  tÃ i khoáº£n root user.

- á»ž gÃ³c trÃªn bÃªn pháº£i cá»§a báº£ng Ä‘iá»u khiá»ƒn, chá»n tÃªn hoáº·c sá»‘ tÃ i khoáº£n cá»§a báº¡n, sau Ä‘Ã³ chá»n Security Credentials (ThÃ´ng tin xÃ¡c thá»±c báº£o máº­t).

- á»ž pháº§n Access keys (KhÃ³a truy cáº­p), chá»n Create access key (Táº¡o khÃ³a truy cáº­p). Náº¿u tÃ¹y chá»n nÃ y khÃ´ng kháº£ dá»¥ng, Ä‘iá»u Ä‘Ã³ cÃ³ nghÄ©a báº¡n Ä‘Ã£ cÃ³ sá»‘ lÆ°á»£ng tá»‘i Ä‘a cÃ¡c khÃ³a truy cáº­p. Báº¡n pháº£i xÃ³a má»™t trong cÃ¡c khÃ³a truy cáº­p hiá»‡n cÃ³ trÆ°á»›c khi táº¡o khÃ³a má»›i. Äá»ƒ biáº¿t thÃªm thÃ´ng tin, xem IAM Object Quotas trong HÆ°á»›ng dáº«n NgÆ°á»i dÃ¹ng IAM.

- TrÃªn trang Alternatives to root user access keys (CÃ¡c phÆ°Æ¡ng Ã¡n thay tháº¿ cho khÃ³a truy cáº­p tÃ i khoáº£n root user), xem xÃ©t cÃ¡c khuyáº¿n nghá»‹ vá» báº£o máº­t. Äá»ƒ tiáº¿p tá»¥c, chá»n vÃ o Ã´ kiá»ƒm vÃ  sau Ä‘Ã³ chá»n Create access key (Táº¡o khÃ³a truy cáº­p).

- TrÃªn trang Retrieve access key (Trang Láº¥y láº¡i khÃ³a truy cáº­p), Access key ID cá»§a báº¡n sáº½ Ä‘Æ°á»£c hiá»ƒn thá»‹.

- DÆ°á»›i má»¥c Secret access key (KhÃ³a truy cáº­p bÃ­ máº­t), chá»n Show (Hiá»ƒn thá»‹) vÃ  sau Ä‘Ã³ sao chÃ©p Access key ID vÃ  Secret key tá»« cá»­a sá»• trÃ¬nh duyá»‡t cá»§a báº¡n, sau Ä‘Ã³ dÃ¡n chÃºng vÃ o má»™t nÆ¡i an toÃ n. TÃ¹y chá»n khÃ¡c, báº¡n cÃ³ thá»ƒ chá»n Download .csv file (Táº£i xuá»‘ng tá»‡p .csv) Ä‘á»ƒ táº£i xuá»‘ng má»™t tá»‡p cÃ³ tÃªn lÃ  rootkey.csv chá»©a Access key ID vÃ  Secret key. LÆ°u tá»‡p á»Ÿ má»™t nÆ¡i an toÃ n.

- Chá»n Done (HoÃ n thÃ nh). Khi báº¡n khÃ´ng cÃ²n cáº§n khÃ³a truy cáº­p ná»¯a, chÃºng tÃ´i khuyáº¿n nghá»‹ báº¡n xÃ³a nÃ³ hoáº·c Ã­t nháº¥t lÃ  xem xÃ©t vÃ´ hiá»‡u hÃ³a nÃ³ Ä‘á»ƒ khÃ´ng ai cÃ³ thá»ƒ láº¡m dá»¥ng.

> LÆ°u Ã½: CÃ¡c bÆ°á»›c nÃ y chá»‰ Ã¡p dá»¥ng cho tÃ i khoáº£n root user trÃªn AWS. Äá»‘i vá»›i cÃ¡c ngÆ°á»i dÃ¹ng IAM hoáº·c vai trÃ² (role), cÃ¡ch táº¡o vÃ  quáº£n lÃ½ khÃ³a truy cáº­p cÃ³ thá»ƒ khÃ¡c nhau.

### XÃ³a má»™t khÃ³a truy cáº­p cho ngÆ°á»i dÃ¹ng gá»‘c trÃªn AWS

#### Quyá»n tá»‘i thiá»ƒu
Äá»ƒ thá»±c hiá»‡n cÃ¡c bÆ°á»›c sau, báº¡n pháº£i cÃ³ Ã­t nháº¥t cÃ¡c quyá»n IAM (Quáº£n lÃ½ Quyá»n vÃ  Danh sÃ¡ch quyá»n) sau Ä‘Ã¢y:

- Báº¡n pháº£i Ä‘Äƒng nháº­p dÆ°á»›i tÆ° cÃ¡ch ngÆ°á»i dÃ¹ng gá»‘c cá»§a tÃ i khoáº£n AWS, Ä‘iá»u nÃ y khÃ´ng yÃªu cáº§u báº¥t ká»³ quyá»n AWS Identity and Access Management (IAM) bá»• sung nÃ o. Báº¡n khÃ´ng thá»ƒ thá»±c hiá»‡n cÃ¡c bÆ°á»›c nÃ y dÆ°á»›i tÆ° cÃ¡ch ngÆ°á»i dÃ¹ng IAM hoáº·c vai trÃ².
- Sá»­ dá»¥ng Ä‘á»‹a chá»‰ email vÃ  máº­t kháº©u cá»§a tÃ i khoáº£n AWS Ä‘á»ƒ Ä‘Äƒng nháº­p vÃ o Báº£ng Ä‘iá»u khiá»ƒn Quáº£n lÃ½ AWS dÆ°á»›i tÆ° cÃ¡ch ngÆ°á»i dÃ¹ng gá»‘c cá»§a báº¡n.
- á»ž gÃ³c trÃªn bÃªn pháº£i cá»§a báº£ng Ä‘iá»u khiá»ƒn, chá»n tÃªn hoáº·c sá»‘ tÃ i khoáº£n cá»§a báº¡n, sau Ä‘Ã³ chá»n **Security Credentials** (Chá»©ng chá»‰ Báº£o máº­t).
- Trong pháº§n **Access keys** (KhÃ³a truy cáº­p), chá»n khÃ³a truy cáº­p mÃ  báº¡n muá»‘n xÃ³a, sau Ä‘Ã³, á»Ÿ má»¥c **Actions** (Thao tÃ¡c), chá»n **Delete** (XÃ³a).

> **LÆ°u Ã½**
> Theo cÃ¡ch khÃ¡c, báº¡n cÃ³ thá»ƒ **Deactivate** (NgÆ°ng hoáº¡t Ä‘á»™ng) má»™t khÃ³a truy cáº­p thay vÃ¬ xÃ³a nÃ³ vÄ©nh viá»…n. Äiá»u nÃ y giÃºp báº¡n cÃ³ thá»ƒ tiáº¿p tá»¥c sá»­ dá»¥ng nÃ³ trong tÆ°Æ¡ng lai mÃ  khÃ´ng cáº§n thay Ä‘á»•i cáº£ ID khÃ³a láº«n khÃ³a bÃ­ máº­t. Trong khi khÃ³a bá»‹ vÃ´ hiá»‡u hÃ³a, má»i yÃªu cáº§u sá»­ dá»¥ng nÃ³ trong cÃ¡c yÃªu cáº§u tá»›i AWS API sáº½ tháº¥t báº¡i vá»›i lá»—i "access denied" (truy cáº­p bá»‹ tá»« chá»‘i).
>
> TrÃªn há»™p thoáº¡i **Delete <access key ID>** (XÃ³a <ID khÃ³a truy cáº­p>), chá»n **Deactivate** (NgÆ°ng hoáº¡t Ä‘á»™ng), nháº­p ID khÃ³a truy cáº­p Ä‘á»ƒ xÃ¡c nháº­n báº¡n muá»‘n xÃ³a nÃ³, sau Ä‘Ã³ chá»n **Delete** (XÃ³a).


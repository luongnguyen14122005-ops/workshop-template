---
title : "KhÃ³a báº£o máº­t U2F"
date :  "2026-10-03" 
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

**Ná»™i dung**
- [KÃ­ch hoáº¡t khÃ³a báº£o máº­t U2F thÃ´ng qua Console](#kÃ­ch-hoáº¡t-khÃ³a-báº£o-máº­t-u2f-thÃ´ng-qua-console)


{{%notice tip%}}
Náº¿u báº¡n khÃ´ng cÃ³ thiáº¿t bá»‹ pháº§n cá»©ng , cÃ³ thá»ƒ bá» qua cÃ¡c thao tÃ¡c dÆ°á»›i Ä‘Ã¢y nhÃ©.
{{%/notice%}}

#### KÃ­ch hoáº¡t khÃ³a báº£o máº­t U2F thÃ´ng qua Console

U2F Security Key lÃ  má»™t giao thá»©c chá»©ng thá»±c má»Ÿ cho phÃ©p ngÆ°á»i dÃ¹ng cÃ³ thá»ƒ truy cáº­p cÃ¡c dá»‹ch vá»¥ trá»±c tuyáº¿p vá»›i má»™t khÃ³a báº£o máº­t duy nháº¥t mÃ  khÃ´ng cáº§n sá»­ dá»¥ng Ä‘áº¿n báº¥t kÃ¬ pháº§n má»m nÃ o.

1. ÄÄƒng nháº­p vÃ o AWS Console.
2. GÃ³c trÃªn bÃªn pháº£i, báº¡n sáº½ tháº¥y tÃªn account cá»§a báº¡n, chá»n vÃ o vÃ  chá»n **My Security Credentials** sau Ä‘Ã³ má»Ÿ rá»™ng Multi-factor authentication (MFA).

3. Äá»ƒ quáº£n lÃ­ khÃ³a báº£o máº­t U2F, báº¡n pháº£i cÃ³ quyá»n tá»« bá»™ quyá»n sau. á»Ÿ thanh bÃªn trÃ¡i, chá»n **Policies** sau Ä‘Ã³ chá»n **Create policy**, chá»n **JSON** tab vÃ  dÃ¡n pháº§n bÃªn dÆ°á»›i vÃ o:

```js
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowManageOwnUserMFA",
            "Effect": "Allow",
            "Action": [
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "DenyAllExceptListedIfNoMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

![MFA](/images/3/0001.png?featherlight=false&width=90pc)

4. Chá»n **Next: Tags**. ÄÃ¢y lÃ  mÃ n hÃ¬nh vá» **Tags** má»™t cÃ´ng cá»¥ dÃ¹ng Ä‘á»ƒ phÃ¢n biá»‡t cÃ¡c tÃ i nguyÃªn cá»§a AWS.
5. Chá»n  **Next: Review**. ÄÃ¢y lÃ  mÃ n hÃ¬nh cho phÃ©p báº¡n review vá» bá»™ quyá»n mÃ  báº¡n Ä‘ang táº¡o ra. 
6. Nháº­p tÃªn bá»™ quyá»n (vÃ­ dá»¥: MFAHardDevice) vÃ  chá»n **Create policy**.

![MFA](/images/3/0002.png?featherlight=false&width=90pc)

![MFA](/images/3/0003.png?featherlight=false&width=90pc)

7. á»ž thanh bÃªn trÃ¡i , chá»n **Dashboard** vÃ  sau Ä‘Ã³ chá»n **Enable MFA**.

![MFA](/images/3/0004.png?featherlight=false&width=90pc)

8. Má»Ÿ rá»™ng Multi-factor authentication (MFA) sau Ä‘Ã³ chá»n **Active MFA**.

9. Trong **Manage MFA Device**, chá»n **U2F security key** sau Ä‘Ã³ nháº¥n **Continue**.
10. Cáº¯m khÃ³a báº£o máº­t U2F vÃ o cá»•ng USB cá»§a mÃ¡y tÃ­nh.

![Image](/images/1-account-setup/U2FSK.png?featherlight=false&width=90pc)

11. Nháº¥n vÃ o khÃ³a báº£o máº­t U2F, vÃ  sau Ä‘Ã³ chá»n **Close** khi U2F thiáº¿t láº­p thÃ nh cÃ´ng.


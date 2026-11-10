---
title : "Thiáº¿t bá»‹ MFA cá»©ng"
date :  "2026-10-03" 
weight : 3
chapter : false
pre : " <b> 2.3 </b> "
---

**Ná»™i dung**

- [KÃ­ch hoáº¡t thiáº¿t bá»‹ MFA pháº§n cá»©ng khÃ¡c thÃ´ng qua Console](#kÃ­ch-hoáº¡t-thiáº¿t-bá»‹-mfa-pháº§n-cá»©ng-khÃ¡c-thÃ´ng-qua-console)

#### KÃ­ch hoáº¡t thiáº¿t bá»‹ MFA pháº§n cá»©ng khÃ¡c thÃ´ng qua Console

1. ÄÄƒng nháº­p vÃ o AWS Console.
2. GÃ³c trÃªn bÃªn pháº£i, báº¡n sáº½ tháº¥y tÃªn account cá»§a báº¡n, chá»n vÃ o vÃ  chá»n **My Security Credentials** sau Ä‘Ã³ má»Ÿ rá»™ng Multi-factor authentication (MFA).

![Image](/images/1-account-setup/MySecurity_v1.png?width=15pc)

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

9. Trong **Manage MFA Device**, chá»n **Other Hardware MFA Device** sau Ä‘Ã³ nháº¥n **Continue**.
10. Nháº­p **Serial Number** á»Ÿ Ä‘áº±ng sau thiáº¿t bá»‹.

![Image](/images/1-account-setup/HardwareMFA.png?featherlight=false&width=90pc)

11. Nháº­p MFA code 1 sau Ä‘Ã³ Ä‘á»£i 30 giÃ¢y vÃ  nháº­p MFA code 2.
12. Chá»n **Assign MFA**.


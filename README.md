# 🛡️ مشروع تأمين البنية السحابية: سحاب اليمن التقنية (Yemen Cloud Security)

## 📋 1. نظرة عامة على المشروع (Project Overview)
يهدف هذا المشروع إلى تصميم وتأمين بنية تحتية سحابية افتراضية لشركة **"سحاب اليمن التقنية"** على منصة **AWS**. تم تطبيق أفضل الممارسات الأمنية العالمية (Security Best Practices) عبر مبدأ "أقل صلاحية ممكنة" (Principle of Least Privilege)، مع بناء طبقات دفاع متعددة تشمل إدارة الهويات والصلاحيات (IAM)، عزل المستخدمين، وفرض المصادقة الثنائية الإجبارية (MFA) للتصدّي لأي محاولات وصول غير مصرح بها.

---

## 👥 2. الهيكل التنظيمي وصلاحيات المستخدمين (IAM Architecture)
لضمان بيئة عمل آمنة، تم تقسيم المستخدمين إلى أدوار تشغيلية محددة بدقة:

* **مدير النظام (`ahmad-admin`):** يمتلك الصلاحيات الإدارية الكاملة لإدارة الموارد والسياسات الأمنية.
* **المطور (`ahmad-dev`):** تم تقييد صلاحياته وتخصيص وصول للقراءة فقط (`AmazonS3ReadOnlyAccess`) لمنع أي تعديلات غير مصرح بها على البيئة التشغيلية.
* **المدقق الأمني (`ahmad-audit`):** مخصص لمراجعة وتقييم السجلات والسياسات دون القدرة على تعديل الموارد.

---

## 🔒 3. سياسة إجبار المصادقة الثنائية (EnforceMFAWithExceptions)
تم إنشاء سياسة أمنية مخصصة تمنع أي مستخدم من تنفيذ عمليات متقدمة ما لم يتم توثيق الحساب بجهاز MFA نشط. إليك كود السياسة (`JSON`):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowViewAccountInfoWithoutMFA",
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy",
                "iam:ListVirtualMFADevices",
                "iam:GetAccountSummary"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowManageMFA",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/*",
                "arn:aws:iam::*:user/$${aws:username}"
            ]
        },
        {
            "Sid": "DenyAllExceptListedIfNoMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary",
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}

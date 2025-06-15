# SkillCooldownUI
```C#
using Sirenix.OdinInspector;
using UnityEngine;
using UnityEngine.UI;

public class SkillCooldownUI : MonoBehaviour
{
    [SerializeField] private Image cooldownFillImage;
    [SerializeField] private float defaultCooldownDuration = 5f; // 기본 쿨다운 시간
    private float cooldownDuration;
    private float cooldownTimer;
    private bool isCoolingDown;

    public void StartCooldown(float duration)
    {
        cooldownDuration = duration;
        cooldownTimer = 0f;
        isCoolingDown = true;
        cooldownFillImage.fillAmount = 0f;
    }

    // 버튼에서 호출할 수 있는 메서드
    [Button]
    public void OnSkillButtonPressed()
    {
        if (!isCoolingDown)
        {
            StartCooldown(defaultCooldownDuration);
        }
    }

    private void Update()
    {
        if (!isCoolingDown) return;

        cooldownTimer += Time.deltaTime;
        float ratio = (cooldownTimer / cooldownDuration);
        cooldownFillImage.fillAmount = Mathf.Clamp01(ratio);

        if (cooldownTimer >= cooldownDuration)
        {
            isCoolingDown = false;
            cooldownFillImage.fillAmount = 0f;
        }
    }
}
```
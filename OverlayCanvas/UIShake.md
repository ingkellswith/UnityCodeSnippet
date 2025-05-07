# UIShake
```C#
using UnityEngine;
using DG.Tweening;

public class UIShakeController : MonoBehaviour
{
    public float punchScale = 1.2f;
    public float punchDuration = 0.4f;

    public float strongShakeDuration = 1.5f;
    public float fadeOutDuration = 1.0f;
    public float maxShakeMagnitude = 20f;

    private RectTransform rectTransform;
    private Vector2 originalAnchoredPos;
    private Vector3 originalScale;

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
        originalAnchoredPos = rectTransform.anchoredPosition;
        originalScale = rectTransform.localScale;
    }

    void Start()
    {
        Shake();
    }

    public void Shake()
    {
        StopAllCoroutines();
        rectTransform.DOKill(true);

        // 1. 확장 (Punch Scale)
        rectTransform.localScale = originalScale;
        rectTransform.DOPunchScale(Vector3.one * punchScale, punchDuration, 10, 1.0f);

        // 2. 강한 흔들림
        StartCoroutine(ShakeRoutine());
    }

    private System.Collections.IEnumerator ShakeRoutine()
    {
        float elapsed = 0f;

        // 강한 흔들림 구간
        while (elapsed < strongShakeDuration)
        {
            Vector2 offset = Random.insideUnitCircle * maxShakeMagnitude;
            rectTransform.anchoredPosition = originalAnchoredPos + offset;

            elapsed += Time.deltaTime;
            yield return null;
        }

        // 감쇠 흔들림 구간
        elapsed = 0f;
        while (elapsed < fadeOutDuration)
        {
            float t = 1f - (elapsed / fadeOutDuration);
            float currentMag = maxShakeMagnitude * t * t; // 부드러운 감쇠 (EaseOut)

            Vector2 offset = Random.insideUnitCircle * currentMag;
            rectTransform.anchoredPosition = originalAnchoredPos + offset;

            elapsed += Time.deltaTime;
            yield return null;
        }

        rectTransform.anchoredPosition = originalAnchoredPos;
        rectTransform.localScale = originalScale;
    }
}
```
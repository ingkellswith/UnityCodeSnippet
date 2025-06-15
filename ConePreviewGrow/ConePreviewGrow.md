# ConePreviewGrow
MeshFilter는 Quad면 됨
```C#
using System.Linq;
using UnityEngine;
using Sirenix.OdinInspector;
using Cysharp.Threading.Tasks;
using System.Threading;
using ZLinq;

[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class ConePreviewGrow : MonoBehaviour
{
    [Title("Cone Settings")]
    public float maxRadius = 3f;
    public float angleDegree = 90f;
    public int segments = 20;
    public float growDuration = 1.0f;

    private float _currentRadius = 0f;
    private float _timer = 0f;
    private MeshFilter _meshFilter;
    private MeshRenderer _meshRenderer;
    private CancellationTokenSource _cts;
    private ConePreviewGrow _childGrower;

    void Start()
    {
        _meshFilter = GetComponent<MeshFilter>();
        _meshFilter.mesh = new Mesh();
        _meshRenderer = GetComponent<MeshRenderer>();
        _meshRenderer.enabled = false;

        // 자식 중 자신(this)이 아닌 ConePreviewGrow 찾아 자동 할당
        _childGrower = GetComponentsInChildren<ConePreviewGrow>()
            .AsValueEnumerable() 
            .FirstOrDefault(x => x != this);
    }

    [Button("Cone Grow")]
    private void StartConeGrow()
    {
        _cts?.Cancel();
        _cts = new CancellationTokenSource();

        GrowConeAsync(_cts.Token).Forget();
        _childGrower?.GrowConeAsync(_cts.Token).Forget();
    }

    public async UniTaskVoid GrowConeAsync(CancellationToken token)
    {
        _currentRadius = 0f;
        _timer = 0f;
        _meshRenderer.enabled = true;

        while (_currentRadius < maxRadius)
        {
            if (token.IsCancellationRequested) return;

            _timer += Time.deltaTime;
            _currentRadius = Mathf.Lerp(0f, maxRadius, _timer / growDuration);
            _meshFilter.mesh = CreateConeMesh(angleDegree, _currentRadius, segments);
            await UniTask.Yield();
        }

        await UniTask.Delay(1000, cancellationToken: token);
        if (!token.IsCancellationRequested)
            _meshRenderer.enabled = false;
    }

    private Mesh CreateConeMesh(float angleDeg, float radius, int segments)
    {
        Mesh mesh = new Mesh();
        Vector3[] vertices = new Vector3[segments + 2];
        int[] triangles = new int[segments * 3];

        vertices[0] = Vector3.zero;
        float angleStep = angleDeg / segments;
        float startAngle = -angleDeg / 2f;

        for (int i = 0; i <= segments; i++)
        {
            float angleRad = Mathf.Deg2Rad * (startAngle + angleStep * i);
            float x = Mathf.Cos(angleRad) * radius;
            float y = Mathf.Sin(angleRad) * radius;
            vertices[i + 1] = new Vector3(x, y, 0f);
        }

        for (int i = 0; i < segments; i++)
        {
            int start = i + 1;
            triangles[i * 3] = 0;
            triangles[i * 3 + 1] = start;
            triangles[i * 3 + 2] = start + 1;
        }

        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.RecalculateNormals();
        return mesh;
    }
}

```
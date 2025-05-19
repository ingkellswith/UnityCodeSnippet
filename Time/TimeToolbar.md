Time Toolbar
===
https://github.com/marijnz/unity-toolbar-extender.git
```C#
using UnityEditor;
using UnityEngine;
using UnityToolbarExtender;

[InitializeOnLoad]
public static class TimeScaleToolbar
{
    private static float timeScale = 1.0f;
    private const float MIN_TIMESCALE = 0f;
    private const float MAX_TIMESCALE = 5f;

    static TimeScaleToolbar()
    {
        ToolbarExtender.RightToolbarGUI.Add(OnToolbarGUI);
    }

    static void OnToolbarGUI()
    {
        GUILayout.FlexibleSpace();

        GUIContent label = new GUIContent("TimeScale");
        GUILayout.Label(label);

        float newTimeScale = GUILayout.HorizontalSlider(Time.timeScale, MIN_TIMESCALE, MAX_TIMESCALE, GUILayout.Width(100));

        if (!Mathf.Approximately(newTimeScale, Time.timeScale))
        {
            Time.timeScale = newTimeScale;
            EditorApplication.QueuePlayerLoopUpdate(); // 변경사항 즉시 반영
            SceneView.RepaintAll(); // 씬뷰 갱신
        }

        GUILayout.Label(Time.timeScale.ToString("F2"), GUILayout.Width(30));
    }
}
```
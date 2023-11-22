![PhoneShopping.gif](https://raw.githubusercontent.com/alanchancl/BlogImg/main/img202311211613347.gif)
# 1.引言
通过Unity3D构建一个手机购买场景，让用户可以从各个角度查看产品、了解详细信息、甚至模拟使用体验的场景。不仅增强了用户的参与感，还提供一个更加直观的购物环境。

# 2、主要类说明

## 2.1 PhoneDetails类
该类为手机详细信息类，包括手机名称、手机模型、手机参数描述和手机在UI界面中对应的按钮四个成员变量。
~~~
public class PhoneDetails
{
    public string Name;//手机名称
    public GameObject phoneModel;//手机模型
    [TextArea(3, 10)] public string Description;//手机参数描述
    public Button modelButton;//对应按钮
}
~~~
以下将举例如何为每款手机创建实例：
![image.png](https://raw.githubusercontent.com/alanchancl/BlogImg/main/img202311211654502.png)

## 2.2 ModelDisplayController类
该类用来管理手机模型的显示和用户交互。
主要方法包括：
~~~
ShowNext() //右滑，展示下一个手机
ShowPrevious()//左滑，展示上一个手机
ShowModel()//展示模型
UpdateModelDisplay()//随着按钮交互，更新模型
UpdateButtonColors()//修改选中按钮的颜色
UpdateDescription()//修改手机端的具体描述
~~~
创建UI界面中下侧的六个交互按钮，然后为交互按钮设置OnClick事件
![image.png](https://raw.githubusercontent.com/alanchancl/BlogImg/main/img202311211719159.png)
为每个按钮设置展示的模型序号，序号从0开始，与Phones数组的顺序一一对应
![image.png](https://raw.githubusercontent.com/alanchancl/BlogImg/main/img202311211723258.png)

## 2.3 SmoothUi3DCamera类
该类用于控制相机，创建一个平滑的用户界面3D相机，允许用户通过鼠标和触摸输入来交互地查看3D场景的对象。
代码参考了Kyle同学的博文[unity 3D模型展示旋转缩放_unity 模型旋转-CSDN博客](https://blog.csdn.net/u013509878/article/details/125294558)

# 3.代码脚本

以下为两个脚本ModelDisplayController.cs和SmoothUi3DCamera.cs的详细代码和注释：
## 3.1 ModelDisplayController.cs
~~~
[System.Serializable]
public class PhoneDetails
{
    public string Name; // 手机名称
    public GameObject phoneModel; // 对应的手机模型
    [TextArea(3, 10)] public string Description; // 手机描述，使用文本区域显示
    public Button modelButton; // 与手机模型相关联的UI按钮
}

public class ModelDisplayController : MonoBehaviour
{
    public PhoneDetails[] phones; // 存储所有手机详情的数组
    public TMPro.TextMeshProUGUI PhoneNameText; // 显示手机名称的UI文本元素
    public TMPro.TextMeshProUGUI descriptionText; // 显示手机描述的UI文本元素
    public Color selectedColor = Color.green; // 选中模型时按钮的颜色
    public Color defaultColor = Color.white; // 默认按钮颜色
    private int currentIndex = -1; // 当前选中的手机模型索引，初始值为-1表示没有选中

    private void Start()
    {
        // 初始化时，更新按钮颜色并默认显示第一个手机模型
        UpdateButtonColors(-1);
        ShowModel(0);
    }

    public void ShowNext()
    {
        // 显示下一个手机模型
        if (phones.Length > 0)
        {
            currentIndex = (currentIndex + 1) % phones.Length;
            ShowModel(currentIndex);
        }
    }

    public void ShowPrevious()
    {
        // 显示前一个手机模型
        if (phones.Length > 0)
        {
            currentIndex--;
            if (currentIndex < 0) currentIndex = phones.Length - 1;
            ShowModel(currentIndex);
        }
    }

    public void ShowModel(int modelIndex)
    {
        // 根据索引显示特定的手机模型
        if (modelIndex >= 0 && modelIndex < phones.Length)
        {
            currentIndex = modelIndex;
            UpdateModelDisplay(currentIndex);
            UpdateButtonColors(currentIndex);
            UpdateDescription(currentIndex); // 更新描述信息
        }
    }

    private void UpdateModelDisplay(int modelIndex)
    {
        // 根据索引激活对应的手机模型
        for (int i = 0; i < phones.Length; i++)
        {
            phones[i].phoneModel.SetActive(i == modelIndex);
        }
    }

    private void UpdateButtonColors(int selectedModelIndex)
    {
        // 更新按钮颜色以反映当前选中的模型
        for (int i = 0; i < phones.Length; i++)
        {
            phones[i].modelButton.image.color = (i == selectedModelIndex) ? selectedColor : defaultColor;
        }
    }

    private void UpdateDescription(int modelIndex)
    {
        // 更新显示的手机名称和描述
        if (modelIndex >= 0 && modelIndex < phones.Length)
        {
            PhoneNameText.text = phones[modelIndex].Name;
            descriptionText.text = phones[modelIndex].Description;
        }
    }
}

~~~
## 3.2 SmoothUi3DCamera.cs
~~~
/// <summary>
/// 挂载到相机上的脚本，用于实现平滑的3D界面相机控制。
/// 支持鼠标和触摸输入，用于在3D环境中查看对象。
/// </summary>

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SmoothUi3DCamera : MonoBehaviour
{
    public Transform pivot; // 相机旋转的中心点
    public Vector3 pivotOffset = Vector3.zero; // 相对于旋转中心点的偏移
    public Transform target; // 相机聚焦的目标对象
    public float distance = 10.0f; // 相机与目标的初始距离
    public float minDistance = 5f; // 相机与目标的最小距离
    public float maxDistance = 15f; // 相机与目标的最大距离
    public float zoomSpeed = 1f; // 缩放速度
    public float xSpeed = 250.0f; // 水平旋转速度
    public float ySpeed = 250.0f; // 垂直旋转速度
    [Header("触摸旋转速度因子")]
    public float touchSpeed = 0.05f; // 触摸旋转的速度因子，经测试0.05f效果较好
    public bool allowYTilt = true; // 是否允许垂直倾斜
    public float yMinLimit = -90f; // 垂直倾斜的最小限制
    public float yMaxLimit = 90f; // 垂直倾斜的最大限制

    private float x = 0.0f; // 内部变量，用于记录x轴旋转
    private float y = 0.0f; // 内部变量，用于记录y轴旋转
    private float targetX = 0f; // 目标x轴旋转值
    private float targetY = 0f; // 目标y轴旋转值
    public float targetDistance = 0f; // 目标距离值
    private float xVelocity = 1f; // x轴旋转的速度
    private float yVelocity = 1f; // y轴旋转的速度
    private float zoomVelocity = 1f; // 缩放的速度

    private Touch oldTouch1;  // 上次触摸点1（手指1）
    private Touch oldTouch2;  // 上次触摸点2（手指2）

    private void Start()
    {
        // 在游戏开始时初始化相机角度和距离
        var angles = transform.eulerAngles;
        // 因为操作的是相机，所以实际操作方向相反
        targetY = x = angles.x;
        targetX = y = ClampAngle(angles.y, yMinLimit, yMaxLimit);
        targetDistance = distance;
    }

    private void LateUpdate()
    {
        // 如果没有指定旋转中心点，则返回
        if (!pivot) return;

        // 鼠标控制区域
        #region 鼠标控制
        // 鼠标中键控制缩放
        var scroll = Input.GetAxis("Mouse ScrollWheel");
        if (scroll > 0.0f) targetDistance -= zoomSpeed;
        else if (scroll < 0.0f) targetDistance += zoomSpeed;
        targetDistance = Mathf.Clamp(targetDistance, minDistance, maxDistance);

        // 鼠标左右键操作旋转
        if (Input.GetMouseButton(0) || (Input.GetMouseButton(1) && (Input.GetKey(KeyCode.LeftControl) || Input.GetKey(KeyCode.RightControl))))
        {
            targetX += Input.GetAxis("Mouse X") * xSpeed * 0.02f;
            if (allowYTilt)
            {
                targetY -= Input.GetAxis("Mouse Y") * ySpeed * 0.02f;
                targetY = ClampAngle(targetY, yMinLimit, yMaxLimit);
            }
        }

        // 平滑过渡到目标旋转和缩放
        x = Mathf.SmoothDampAngle(x, targetX, ref xVelocity, 0.3f);
        y = allowYTilt ? Mathf.SmoothDampAngle(y, targetY, ref yVelocity, 0.3f) : targetY;
        Quaternion rotation = Quaternion.Euler(y, x, 0);
        distance = Mathf.SmoothDamp(distance, targetDistance, ref zoomVelocity, 0.5f);
        Vector3 position = rotation * new Vector3(0.0f, 0.0f, -distance) + pivot.position + pivotOffset;
        transform.rotation = rotation;
        transform.position = position;
        #endregion

        // 触摸控制区域
        #region 触摸控制
        // 单点触摸，水平和垂直旋转
        if (1 == Input.touchCount)
        {
            Touch touch = Input.GetTouch(0);
            Vector2 deltaPos = touch.deltaPosition;
            targetX += deltaPos.x * xSpeed * 0.02f * touchSpeed;
            if (allowYTilt)
            {
                targetY -= deltaPos.y * ySpeed * 0.02f * touchSpeed;
                targetY = ClampAngle(targetY, yMinLimit, yMaxLimit);
            }
        }

        // 双点触摸，放大缩小
        if (2 == Input.touchCount)
        {
            Touch newTouch1 = Input.GetTouch(0);
            Touch newTouch2 = Input.GetTouch(1);
            // 如果是新的触摸点，则只记录，不处理
            if (newTouch2.phase == TouchPhase.Began)
            {
                oldTouch2 = newTouch2;
                oldTouch1 = newTouch1;
                return;
            }

            // 计算两次触摸之间的距离变化，用于缩放
            float oldDistance = Vector2.Distance(oldTouch1.position, oldTouch2.position);
            float newDistance = Vector2.Distance(newTouch1.position, newTouch2.position);
            float offset = newDistance - oldDistance;
            if (offset > 0.0f) targetDistance -= zoomSpeed;
            else if (offset < 0.0f) targetDistance += zoomSpeed;
            targetDistance = Mathf.Clamp(targetDistance, minDistance, maxDistance);

            // 更新触摸点记录
            oldTouch1 = newTouch1;
            oldTouch2 = newTouch2;
        }
        #endregion
     
    }

    /// <summary>
    /// 限制角度在指定范围内。
    /// </summary>
    /// <param name="angle">要限制的角度</param>
    /// <param name="min">最小角度</param>
    /// <param name="max">最大角度</param>
    /// <returns>限制后的角度</returns>
    private static float ClampAngle(float angle, float min, float max)
    {
        if (angle < -360) angle += 360;
        if (angle > 360) angle -= 360;
        return Mathf.Clamp(angle, min, max);
    }
}

~~~

# 4.资源获取



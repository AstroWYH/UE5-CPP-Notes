```C#
    void Start()
    {
        myButton = GameObject.Find("myBtn").GetComponent<Button>();
        myButton.onClick.AddListener(FindAndLogTestScripts);

        Debug.Log("[Async] StartAsync 1");
        StartAsync().Forget();
        Debug.Log("[Async] StartAsync 2");
    }

    async UniTask StartAsync()
    {
        Debug.Log("[Async] Before Func1");
        Func1();
        Debug.Log("[Async] After Func1");
        await Func2();
        Debug.Log("[Async] Before Func3");
        Func3();
        Debug.Log("[Async] After Func3");
    }

    void Func1()
    {
        Debug.Log("[Async] Func1");
    }

    async UniTask Func2()
    {
        Debug.Log("[Async] Func2");
        await UniTask.Delay(1000); // 添加实际的异步操作
        Debug.Log("[Async] Func2 1000ms");
    }

    void Func3()
    {
        Debug.Log("[Async] Func3");
    }
```

![image](https://github.com/user-attachments/assets/09c9ee10-4cf7-4092-93a1-b939be347791)

![image](https://github.com/user-attachments/assets/65a7c8d8-c165-4283-8b92-b01be9b4c52a)

结论1：主线程Start()里，不能调用await，可以开启异步调.Forget()。只有在async方法里，才能调await
结论2：在走await Func2();时，[Async] Before Func3会阻塞，但主线程的[Async] StartAsync End不会阻塞
结论3：在走Func4().Forget();时，[Async] After Func4不会阻塞
结论2和结论3：await的下一句会阻塞，Forget的下一句不会阻塞
```C#
    void Start()
    {
        Debug.Log("[Async] StartAsync Start");
        StartAsync().Forget();
        Debug.Log("[Async] StartAsync End");
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
        Func4().Forget();
        Debug.Log("[Async] After Func4");
    }

    async UniTask Func2()
    {
        Debug.Log("[Async] Func2 Before 1000ms");
        await UniTask.Delay(1000);
        Debug.Log("[Async] Func2 After 1000ms");
    }

    async UniTask Func4()
    {
        Debug.Log("[Async] Func4 Before 1000ms");
        await UniTask.Delay(1000);
        Debug.Log("[Async] Func4 After 1000ms");
    }
    void Func1()
    {
        Debug.Log("[Async] Func1");
    }

    void Func3()
    {
        Debug.Log("[Async] Func3");
    }
```

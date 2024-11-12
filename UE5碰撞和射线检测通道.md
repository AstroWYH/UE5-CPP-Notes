## 前置基础知识
![image](https://github.com/user-attachments/assets/c86156f9-c54c-40e9-843a-492921fe0f8b)
![image](https://github.com/user-attachments/assets/e981ffcd-e377-4e77-897e-3e4f49395229)

![image](https://github.com/user-attachments/assets/7185aebc-0c54-4022-87b6-807449ba9e10)
![image](https://github.com/user-attachments/assets/1d68315f-b54b-4e6b-9d27-da05ffba5564)
![image](https://github.com/user-attachments/assets/5d5850fa-44fa-4991-9d61-929f7796e350)


## 0 首先需要新增Object种类和射线种类，以MyTrace和Grass/Tree为例
![image](https://github.com/user-attachments/assets/71639978-8f8f-40a2-841b-1a4581285831)

## 1 通过Channel检测，则只需要设置Object对射线的响应，Tree对射线是block Grass对射线是overlap
![image](https://github.com/user-attachments/assets/41462540-e6b4-4e56-9448-3ba5fe498dbb)
![image](https://github.com/user-attachments/assets/d0d7b026-8b2d-4c31-b885-8f158f974758)
![image](https://github.com/user-attachments/assets/0368e032-b9ff-405f-80f3-7b7e7c193132)
![image](https://github.com/user-attachments/assets/d76a154d-eb39-4a62-a80a-102cd57b0acd)

## 2 通过Object检测，则不需要管右侧Object的配置，只需要关注调用接口时传递的Object种类
![image](https://github.com/user-attachments/assets/b321e2df-5eff-4ea7-9742-8a719db6f0fa)
![image](https://github.com/user-attachments/assets/8771c30b-bab8-40d5-8d90-6899c89c8875)

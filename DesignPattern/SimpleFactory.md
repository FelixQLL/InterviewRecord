# 使用简单工厂模式封装UI组件

## 定义抽象基类
``` C++
class ControlBase;
using ControlBasePtr = boost::shared_ptr<ControlBase>;
class ControlBase : Generable {
 public:
  ControlBase() {}
  virtual ~ControlBase() {}
  virtual Json::Value GenerateJsonValue() = 0;
};
```

## 实现具体类
``` C++
class Text : public ControlBase {
public:
    void GenerateJsonValue() override {
        std::cout << "Text Component" << std::endl;
    }
};

class Button : public ControlBase {
public:
    void GenerateJsonValue() override {
        std::cout << "Button Component" << std::endl;
    }
};
```

## 定义工厂类
``` C++
class ControlFactory : public common::Singleton<ControlFactory> {
 public:
  Json::Value GenerateComboboxJsonValue(const std::string& namesp, const int32_t& seleted_index,
                                        std::vector<ContainerControl::Item>& items);

  Json::Value GenerateRadioButtonJsonValue(const std::string& name, const std::string& link_field, int32_t select_index,
                                           std::vector<Text> items, Binding<bool> is_enabled, Binding<bool> is_visible,
                                           int32_t recommend_index = -1);
  Json::Value GenerateScrollerBarJsonValue(const std::string& name, const std::string& link_field,
                                           const std::string& value, Binding<bool> is_enabled,
                                           Binding<bool> is_visible);
  Json::Value GenerateTabJsonValue(const std::string& name, const std::string& link_field, int32_t select_index,
                                   std::vector<Text> items, Binding<bool> is_enabled, Binding<bool> is_visible);
};
```

## 注册组件并使用工厂类创建组件
``` C++
int main() {
ControlFactory factory;
factory.create();
```
工厂模式继承单例模式
``` C++
int main() {
auto json_tab = ControlFactory::Instance().GenerateTabJsonValue(name, link_field, selected_index, vec_text,
                                                                    enable_bind, Binding<bool>());
```

## 工厂类继承单例模式优缺点
1. 保证工厂类**全局唯一**
2. **简化访问**，直接通过单例Instance访问
3. **资源节约**，避免了多次创建和销毁工厂对象
4. **状态共享**，所有调用者共享同一个 ControlFactory 实例。这意味着可以在工厂类中保存一些全局状态或配置信息，所有调用者都可以访问和修改这些状态。


## 工厂模式使用场景
1. 当产品的创建逻辑较为复杂或者频繁变化，而使用工厂模式可以将创建逻辑封装起来；
2. 当需要创建一系列具有某些共性但行为不同的对象时，可以使用抽象工厂模式来提供一个创建对象的接口，客户端通过这个接口创建对象，而不需要知道具体的对象类。

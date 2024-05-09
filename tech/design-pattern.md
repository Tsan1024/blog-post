---
title:       "设计模式"
subtitle:    ""
description: ""
date:        2023-12-27T23:40:25+08:00
author:      ""
image:       ""
tags:        ["design-pattern", "golang"]
categories:  ["Tech" ]
---
# 引言

## 创见型模式

+ 工厂方法模式
+ 抽象工厂模式
+ 生成器模式
+ 原型模式
+ 单例模式

## 结构型模式

+ 适配器模式
+ 桥接模式
+ 组合模式
+ 装饰模式
+ 外观模式
+ 享元模式
+ 代理模式

## 行为模式

+ 责任链模式
+ 命令模式
+ 迭代器模式
+ 中介者模式
+ 备忘录模式
+ 观察者模式
+ 状态模式
+ 策略模式
+ 模版方法模式
+ 访问者模式
+ 

## 简单工厂模式，工厂模式，抽象工厂模式

简单工厂：唯一工厂类，一个产品抽象类，工厂类的创建方法依据入参判断并创建具体产品对象。
工厂方法：多个工厂类，一个产品抽象类，利用多态创建不同的产品对象，避免了大量的if-else判断。
抽象工厂：多个工厂类，多个产品抽象类，产品子类分组，同一个工厂实现类创建同组中的不同产品，减少了工厂子类的数量。

### simple factory

```
# 简单工厂
package factory

import "fmt"

type Product interface {
	Show(name string)
}

type ProductA struct {
	Name string
}

type ProductB struct {
	Name string
}

func (p *ProductA) Show(name string) {
	fmt.Println("ProductA ", p.Name)
}

func (p *ProductB) Show(name string) {
	fmt.Println("ProductB ", p.Name)
}

type Factory struct {
}

func (f Factory) NewSimpleFactory(t string) Product {
	switch t {
	case "A":
		return &ProductA{Name: t}
	case "B":
		return &ProductB{Name: t}
	}
	return nil
}
```

```
# 简单工厂测试
package factory

import (
	"reflect"
	"testing"
)

func TestNewSimpleFactory(t *testing.T) {
	type args struct {
		t string
	}
	tests := []struct {
		name string
		args args
		want Product
	}{
		{
			name: "A",
			args: args{t: "A"},
			want: &ProductA{Name: "A"},
		},
		{
			name: "B",
			args: args{t: "B"},
			want: &ProductB{Name: "B"},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			f := Factory{}
			if got := f.NewSimpleFactory(tt.args.t); !reflect.DeepEqual(got, tt.want) {
				t.Errorf("NewFactory() = %v, want %v", got, tt.want)
			}
		})
	}
}

```

```
## c++ implement
#include <iostream>
#include <string>

class Product {
public:
    Product(std::string name): Name(name){}
    std::string GetName() {
        return this->Name;
    }
    virtual void Show(const std::string& name) = 0;
private:
    std::string Name;
};

class ProductA : public Product {
public:
    using Product::Product;
    void Show(const std::string& name) override {
        std::cout << "ProductA " << GetName() << std::endl;
    }
};

class ProductB : public Product {
public:
    using Product::Product;
    void Show(const std::string& name) override {
        std::cout << "ProductB " << GetName() << std::endl;
    }
};

class SimpleFactory {
public:
    Product* NewSimpleProduct(const std::string& t) {
        if (t == "A") {
            return new ProductA(t);
        } else if (t == "B") {
            return new ProductB(t);
        }
        return nullptr;
    }
};

int main() {
    SimpleFactory factory;

    Product* productA = factory.NewSimpleProduct("A");
    if (productA) {
        productA->Show("A");
        delete productA;
    }

    Product* productB = factory.NewSimpleProduct("B");
    if (productB) {
        productB->Show("B");
        delete productB;
    }

    return 0;
}

```


### factory pattern


```
#include <iostream>
#include <string>

// Product interface
class Product {
public:
    virtual void Show() const = 0;
    virtual ~Product() = default;
};

// ProductA class
class ProductA : public Product {
public:
    void Show() const override {
        std::cout << "ProductA" << std::endl;
    }
};

// ProductB class
class ProductB : public Product {
public:
    void Show() const override {
        std::cout << "ConcreteProductB" << std::endl;
    }
};

// Factory interface
class Factory {
public:
    virtual Product* CreateProduct() const = 0;
    virtual ~Factory() = default;
};

// FactoryA class
class FactoryA : public Factory {
public:
    Product* CreateProduct() const override {
        return new ProductA();
    }
};

// FactoryB class
class FactoryB : public Factory {
public:
    Product* CreateProduct() const override {
        return new ProductB();
    }
};

int main() {
    // Create FactoryA
    Factory* factoryA = new FactoryA();
    Product* productA = factoryA->CreateProduct();
    productA->Show();

    delete productA;
    delete factoryA;

    // Create FactoryB
    Factory* factoryB = new FactoryB();
    Product* productB = factoryB->CreateProduct();
    productB->Show();

    delete productB;
    delete factoryB;

    return 0;
}

```

### 抽象工厂

```
# include <iostream>

class AbstractProductA {
public:
    virtual void ShowA() const = 0;
    virtual ~AbstractProductA() = default; 
};

class AbstractProductB {
public:   
    virtual void ShowB() const = 0;
    virtual ~AbstractProductB() = default; 
};

class ProductA1 : public AbstractProductA {
public:   
    void ShowA() const override{
        std::cout << "Product A" << std::endl;
    }
};

class ProductB1 : public AbstractProductB {
public:   
    void ShowB() const override{
        std::cout << "Product B" << std::endl;
    }
};

class AbstractFactory {
public:   
    virtual AbstractProductA* CreateProductA() const = 0;
    virtual AbstractProductB* CreateProductB() const = 0;
    virtual ~AbstractFactory() = default; 
};

class AbstractFactory1 : public AbstractFactory {
public:   
    AbstractProductA* CreateProductA() const override {
        return new ProductA1();
    }

    AbstractProductB* CreateProductB() const override {
        return new ProductB1();
    }
};

int main() {
    AbstractFactory *factory = new AbstractFactory1();
    AbstractProductA* Product1 = factory->CreateProductA();
    AbstractProductB* Product2 = factory->CreateProductB();
    Product1->ShowA();
    Product2->ShowB();

    delete Product1;
    delete Product2;
    delete factory;
    return 0;
}
```

参考资料

1. https://lailin.xyz/post/go-design-pattern.html
2. https://refactoringguru.cn/design-patterns
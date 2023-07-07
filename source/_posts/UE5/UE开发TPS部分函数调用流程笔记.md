---
title: {{ title }}
date: {{ date }}
poc: true
categories: 
- [开发, GamePlay]
tags: [C++, UE, UnrealEngine, GamePlay]
---

1. 在子弹类里面编写一个OnProjectileImpact函数用作回调函数
```cpp
void AMyProjectile::OnProjectileImpact(UPrimitiveComponent* HitComponent, AActor* OtherActor,
                                       UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor)
	{
		UGameplayStatics::ApplyPointDamage(OtherActor, Damage, NormalImpulse, Hit, GetInstigator()->Controller, this,
		                                   DamageType);//施加伤害的函数是UE自己实现的, 我们只需要调用
                                           //后续会提到这个函数的调用逻辑
	}

	Destroy();
}
```

2. 在子弹类的构造函数里面绑定上述回调函数, 需要注意的是 只有服务端可以运行这段逻辑 因此需要检查一下
```cpp
if (GetLocalRole() == ROLE_Authority)
	{
		SphereComponent->OnComponentHit.AddDynamic(this, &ThisClass::OnProjectileImpact);
	}

```
这样当子弹被碰撞的时候就可以调用这个回调函数了

3. 看UE源码`UGameplayStatics::ApplyPointDamage` 可以发现`return DamagedActor->TakeDamage(BaseDamage, PointDamageEvent, EventInstigator, DamageCauser);` 这一行
也就是说这个代码的伤害实现是基于`DamagedActor`对象的`TakeDamage`方法
我们去人物类那边
```cpp
float Amutiplayer_demoCharacter::TakeDamage(float DamageTaken, FDamageEvent const& DamageEvent,
                                            AController* EventInstigator, AActor* DamageCauser)
{
	float damageApplied = CurrentHealth - DamageTaken;
	//need a lock here, or network-sync not insurance if float calc slower that network latency and client CPU disorder execute.
	SetCurrentHealth(damageApplied);
	return damageApplied;
}
```
这个方法在父类里面UE已经提供了虚接口, 我们需要重写, 如果不考虑防御值等效果, 我们直接setHealth就可以了


整个代码合起来,调用图如下:
`StartFire()`->`HandleFire()`->RFC: `HandleFire_Implementation()`->`GetWorld()->SpawnActor<AMyProjectile>`->`AMyProjectile` constructor set `bReplicates` `true`
->collision happen->`OnProjectileImpact()`->`ApplyPointDamage()`->`DamagedActor->AActor::TakeDamage()`->virtual->`Amutiplayer_demoCharacter::TakeDamage()`
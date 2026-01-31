---
title: "The Case Against Generics"
date: 2025-01-06
tags: ["abstraction"]
draft: true
---

Generics are often hailed as the pinnacle of type safety in modern programming.
They allow us to write reusable, type-checked code that works across multiple
data structures. However, in many enterprise systems, generics are grossly
overused for problems that are better solved with simple polymorphism. This
leads to the **Illusion of Type Safety**, where the type system becomes a source
of boilerplate rather than a source of protection.

## The Illusion of Type Safety

Consider a generic interface for a `Blobstore` that is parameterised by its
region name:

```java
public interface Blobstore<R extends RegionName> {
    R getRegion();
    void upload(Path path);
}
```

This seems like a strong design. By using a generic parameter `R`, we are
explicitly linking the blobstore to its specific region type (e.g.,
`AwsBlobstore` with `AwsRegionName`). But look at how this abstraction is
actually used in the system:

```java
public void process(List<Blobstore<?>> blobstores) {
    for (Blobstore<?> b : blobstores) {
        RegionName region = b.getRegion();
        // ...
    }
}
```

The "type safety" provided by the generic is an illusion. Because we want to
handle blobstores agnostically, we are forced to use the wildcard `?`. At the
call site, `b.getRegion()` returns the base `RegionName` anyway. The only way to
get the specific `AwsRegionName` is to already know you have an `AwsBlobstore`
and perform a cast. We haven't gained any safety; we've only added boilerplate
to every reference of the type.

## The Resolution: Covariant Return Types

We can achieve true, usable type safety by removing the generics and leveraging
**Covariant Return Types**. In Java, an overriding method can return a more
specific type than the method it overrides.

```java
public interface Blobstore {
    RegionName getRegion();
    void upload(Path path);
}

// ...

public record AwsBlobstore(...) implements Blobstore {
    @Override
    public AwsRegionName getRegion() {  // covariant return type
        return awsRegionName;
    }
}
```

This design is vastly superior. The interface remains clean and easy to use
agnostically. However, if you are working specifically with an `AwsBlobstore`,
you get the specific `AwsRegionName` without any casting or generic noise. You
get the same level of type safety, but it is available when you need it and
stays out of the way when you don't.

## The Synthesis

Why are generics the wrong abstraction here? Because we are dealing with a
**Polymorphism Problem**, not a **Container Problem**. Generics are designed for
containers (like `List<T>`) where the container itself doesn't care about the
type, but the caller does.

When you use generics for a hierarchy like `Blobstore`, you are fighting the
type system. You are forcing every consumer to understand and maintain the
generic signature, even when they only care about the base interface. This leads
to "generic creep," where a single type parameter cascades through your entire
service layer, adding complexity without adding value.

## The Insight

Don't let the allure of generics blind you to simpler solutions. If you find
yourself using `Wildcards` everywhere or suppressing "raw type" warnings, your
abstraction is likely broken. Favor simple interfaces and covariant return types
for hierarchy problems. Let your objects define their own specific types rather
than forcing the type system to do it for you.

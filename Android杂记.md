---
title: Android杂记
date: 2016-12-17 10:32:45
tags: 移动端
categories: Android
---
## 动画
```Java
  // ShareElemt
Intent intent = new Intent(this, SecondShareElemActivity.class);
        Pair first = new Pair<>(firstSharedView, ViewCompat.getTransitionName(firstSharedView));
        Pair second = new Pair<>(secondSharedView, ViewCompat.getTransitionName(secondSharedView));

        ActivityOptionsCompat transitionActivityOptions =
                ActivityOptionsCompat.makeSceneTransitionAnimation(
                        this, first, second);

        ActivityCompat.startActivity(this,
            intent, transitionActivityOptions.toBundle());
```

```Java
// Circular
secondView.setVisibility(View.VISIBLE);
        int centerX = (v.getLeft() + v.getRight()) / 2;
        int centerY = (v.getTop() + v.getBottom()) / 2;
        float finalRadius = (float) Math.hypot((double) centerX, (double) centerY);
        Animator mCircularReveal = ViewAnimationUtils.createCircularReveal(
                secondView, centerX, centerY, 0, finalRadius);

        mCircularReveal.setDuration(400).start();
```

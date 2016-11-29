---
title: Android事件
date: 2015-08-18 12:08:59
categories:
- 开发
- Android
tags:
- Android
---

开发中事件冲突处理，需要更好的理解清除View事件机制。还是看源码，View和View Group中几个方法就可以
<!--more-->

## View
ViewGroup 一般为布局 可以addView 本身也是View的子类
onInterceptTouchEvent() 为ViewGroup中方法

##View事件分发

### Activity到RootView传递
Activity(持有Window引用，Window唯一实现PhoneWindow)调用 ondispatchTouchEvent()
```
/**
 * Called to process touch screen events.  You can override this to
 * intercept all touch screen events before they are dispatched to the
 * window.  Be sure to call this implementation for touch screen events
 * that should be handled normally.
 *
 * @param ev The touch screen event.
 *
 * @return boolean Return true if this event was consumed.
 */
public boolean dispatchTouchEvent(MotionEvent ev) {
  　// Activity类中
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    // getWindow获取PhoneWindow实例，调用该类中superDispatchTouchEvent()
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```
PhoneWindow(Window实现，持有DecorView实例) 调用superDispatchTouchEvent()
```
//PhoneWindow类中
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    // 分发事件到DecorView
    return mDecor.superDispatchTouchEvent(event);
}
```
DecorView (顶级View 继承自FrameLayout。包含一个布局生成的View，参见PhoneWindow中generateLayout(),
其中存在一个id为content的FrameLayout,作为setContentView的父容器)调用superDispatchTouchEvent()---->
```
// DecorView中 调用到ViewGroup中
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```
### ViewGroup View 事件分发
ViewGroup(DecorView继承自FrameLayout，FrameLayout继承自ViewGroup,故会调用至这里) 调用dispatchTouchEvent()简要流程：
判断是否拦截，会调用onInterceptTouchEvent()
如果决定拦截事件，后续事件不会在调用onInterceptTouchEvent,直接拦截
继续调用dispatchTransformedTouchEvent，其中参数child传null，在该方法中child==null ->
调用到View类中dispatchTouchEvent super.dispatchTouchEvent(event), child!=null->
调用到child中dispatchTouchEvent(event)
```

  // ViewGroup 事件拦截
  public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
            && ev.getAction() == MotionEvent.ACTION_DOWN
            && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
            && isOnScrollbarThumb(ev.getX(), ev.getY())) {
        return true;
    }
    return false;
  }

  /**
   * 最终从dispatchTouchEvent调用过来的分发方法
   * Transforms a motion event into the coordinate space of a particular child view,
   * filters out irrelevant pointer ids, and overrides its action if necessary.
   * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
   */
  private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
          View child, int desiredPointerIdBits) {
      final boolean handled;

      // Canceling motions is a special case.  We don't need to perform any transformations
      // or filtering.  The important part is the action, not the contents.
      final int oldAction = event.getAction();
      if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
          event.setAction(MotionEvent.ACTION_CANCEL);
          if (child == null) {
              // 调用到View
              handled = super.dispatchTouchEvent(event);
          } else {
              //调用子View的dispatchTouchEvent，如果子View 是ViewGroup 则和现在这样流程一致如果子View是View 则走View分发流程
              handled = child.dispatchTouchEvent(event);
          }
          event.setAction(oldAction);
          return handled;
      }

      // Calculate the number of pointers to deliver.
      final int oldPointerIdBits = event.getPointerIdBits();
      final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

      // If for some reason we ended up in an inconsistent state where it looks like we
      // might produce a motion event with no pointers in it, then drop the event.
      if (newPointerIdBits == 0) {
          return false;
      }

      // If the number of pointers is the same and we don't need to perform any fancy
      // irreversible transformations, then we can reuse the motion event for this
      // dispatch as long as we are careful to revert any changes we make.
      // Otherwise we need to make a copy.
      final MotionEvent transformedEvent;
      if (newPointerIdBits == oldPointerIdBits) {
          if (child == null || child.hasIdentityMatrix()) {
              if (child == null) {
                  handled = super.dispatchTouchEvent(event);
              } else {
                  final float offsetX = mScrollX - child.mLeft;
                  final float offsetY = mScrollY - child.mTop;
                  event.offsetLocation(offsetX, offsetY);

                  handled = child.dispatchTouchEvent(event);

                  event.offsetLocation(-offsetX, -offsetY);
              }
              return handled;
          }
          transformedEvent = MotionEvent.obtain(event);
      } else {
          transformedEvent = event.split(newPointerIdBits);
      }

      // Perform any necessary transformations and dispatch.
      if (child == null) {
          handled = super.dispatchTouchEvent(transformedEvent);
      } else {
          final float offsetX = mScrollX - child.mLeft;
          final float offsetY = mScrollY - child.mTop;
          transformedEvent.offsetLocation(offsetX, offsetY);
          if (! child.hasIdentityMatrix()) {
              transformedEvent.transform(child.getInverseMatrix());
          }

          handled = child.dispatchTouchEvent(transformedEvent);
      }

      // Done.
      transformedEvent.recycle();
      return handled;
  }

// ViewGroup 事件分发
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // 判断是否需要拦截
            // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    // 调用到ViewGroup的onInterceptTouchEvent() 返回true 拦截 false 不拦截
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

            // Update list of touch targets for pointer down, if needed.
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
            if (!canceled && !intercepted) {

                // If the event is targeting accessiiblity focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            //分发到child
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // 没有子View或者拦截事件
            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        // 分发事件到child
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```
### View事件分发
View中dispatchTouchEvent处理 先判断有无OnTouchListener 有 该方法处理事件 返回true 分发结束 没有交给 onTouchEvent中处理View的事件如 点击 长按 等
```
/**
   * Pass the touch screen motion event down to the target view, or this
   * view if it is the target.
   *
   * @param event The motion event to be dispatched.
   * @return True if the event was handled by the view, false otherwise.
   */
  public boolean dispatchTouchEvent(MotionEvent event) {
      // If the event should be handled by accessibility focus first.
      if (event.isTargetAccessibilityFocus()) {
          // We don't have focus or no virtual descendant has it, do not handle the event.
          if (!isAccessibilityFocusedViewOrHost()) {
              return false;
          }
          // We have focus and got the event, then use normal event dispatch.
          event.setTargetAccessibilityFocus(false);
      }

      boolean result = false;

      if (mInputEventConsistencyVerifier != null) {
          mInputEventConsistencyVerifier.onTouchEvent(event, 0);
      }

      final int actionMasked = event.getActionMasked();
      if (actionMasked == MotionEvent.ACTION_DOWN) {
          // Defensive cleanup for new gesture
          stopNestedScroll();
      }

      if (onFilterTouchEventForSecurity(event)) {
          if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
              result = true;
          }
          //noinspection SimplifiableIfStatement
          ListenerInfo li = mListenerInfo;
          // 判断OnTouchListener
          if (li != null && li.mOnTouchListener != null
                  && (mViewFlags & ENABLED_MASK) == ENABLED
                  && li.mOnTouchListener.onTouch(this, event)) {
              result = true;
          }
          // 交给onTouchEvent处理
          if (!result && onTouchEvent(event)) {
              result = true;
          }
      }

      if (!result && mInputEventConsistencyVerifier != null) {
          mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
      }

      // Clean up after nested scrolls if this is the end of a gesture;
      // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
      // of the gesture.
      if (actionMasked == MotionEvent.ACTION_UP ||
              actionMasked == MotionEvent.ACTION_CANCEL ||
              (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
          stopNestedScroll();
      }

      return result;
  }

  public boolean onTouchEvent(MotionEvent event) {
          final float x = event.getX();
          final float y = event.getY();
          final int viewFlags = mViewFlags;
          final int action = event.getAction();

          if ((viewFlags & ENABLED_MASK) == DISABLED) {
              if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                  setPressed(false);
              }
              // A disabled view that is clickable still consumes the touch
              // events, it just doesn't respond to them.
              return (((viewFlags & CLICKABLE) == CLICKABLE
                      || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                      || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
          }
          if (mTouchDelegate != null) {
              if (mTouchDelegate.onTouchEvent(event)) {
                  return true;
              }
          }

          if (((viewFlags & CLICKABLE) == CLICKABLE ||
                  (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                  (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
              switch (action) {
                  case MotionEvent.ACTION_UP:
                      boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                      if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                          // take focus if we don't have it already and we should in
                          // touch mode.
                          boolean focusTaken = false;
                          if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                              focusTaken = requestFocus();
                          }

                          if (prepressed) {
                              // The button is being released before we actually
                              // showed it as pressed.  Make it show the pressed
                              // state now (before scheduling the click) to ensure
                              // the user sees it.
                              setPressed(true, x, y);
                         }

                          if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                              // This is a tap, so remove the longpress check
                              removeLongPressCallback();

                              // Only perform take click actions if we were in the pressed state
                              if (!focusTaken) {
                                  // Use a Runnable and post this rather than calling
                                  // performClick directly. This lets other visual state
                                  // of the view update before click actions start.
                                  if (mPerformClick == null) {
                                      mPerformClick = new PerformClick();
                                  }
                                  if (!post(mPerformClick)) {
                                      performClick();
                                  }
                              }
                          }

                          if (mUnsetPressedState == null) {
                              mUnsetPressedState = new UnsetPressedState();
                          }

                          if (prepressed) {
                              postDelayed(mUnsetPressedState,
                                      ViewConfiguration.getPressedStateDuration());
                          } else if (!post(mUnsetPressedState)) {
                              // If the post failed, unpress right now
                              mUnsetPressedState.run();
                          }

                          removeTapCallback();
                      }
                      mIgnoreNextUpEvent = false;
                      break;

                  case MotionEvent.ACTION_DOWN:
                      mHasPerformedLongPress = false;

                      if (performButtonActionOnTouchDown(event)) {
                          break;
                      }

                      // Walk up the hierarchy to determine if we're inside a scrolling container.
                      boolean isInScrollingContainer = isInScrollingContainer();

                      // For views inside a scrolling container, delay the pressed feedback for
                      // a short period in case this is a scroll.
                      if (isInScrollingContainer) {
                          mPrivateFlags |= PFLAG_PREPRESSED;
                          if (mPendingCheckForTap == null) {
                              mPendingCheckForTap = new CheckForTap();
                          }
                          mPendingCheckForTap.x = event.getX();
                          mPendingCheckForTap.y = event.getY();
                          postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                      } else {
                          // Not inside a scrolling container, so show the feedback right away
                          setPressed(true, x, y);
                          checkForLongClick(0, x, y);
                      }
                      break;

                  case MotionEvent.ACTION_CANCEL:
                      setPressed(false);
                      removeTapCallback();
                      removeLongPressCallback();
                      mInContextButtonPress = false;
                      mHasPerformedLongPress = false;
                      mIgnoreNextUpEvent = false;
                      break;

                  case MotionEvent.ACTION_MOVE:
                      drawableHotspotChanged(x, y);

                      // Be lenient about moving outside of buttons
                      if (!pointInView(x, y, mTouchSlop)) {
                          // Outside button
                          removeTapCallback();
                          if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                              // Remove any future long press/tap checks
                              removeLongPressCallback();

                              setPressed(false);
                          }
                      }
                      break;
              }

              return true;
          }

          return false;
      }
```

## 总结
所有的控件都是ViewGroup套View或ViewGroup，其事件分发如果没有特殊实现，会按照其父类ViewGroup或者View来分发
搞清楚了这两个，无论如何嵌套，其分发都是很简单的
分发流程从外到里,事件消费从里到外，最终会到Activity的分发方法中，没有子View处理事件则默认执行Activity中onTouchEvent

很多文章都会总结各种执行流程，如果自己看下这几个方法，了解清楚其原理，根本不需要记住哪些流程，各种流程也是从着基本的推出来的，并没有想象的那么难理解，包括事件冲突解决方式也是相关处理

沉淀而不是浮于表面

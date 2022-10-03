## CLEAN CODE LÀ GÌ?
Clean code dịch theo nghĩa đen là "mã nguồn sạch". Vậy như thế nào là mã nguồn sạch? Ở bài viết này chúng ta gói gọn trong phạm vi là typescript. Theo góc nhìn của người viết. Để code sạch thì:
- Tên class, function phải rõ nghĩa, đọc vào là biết nó là gì, làm gì. Ví dụ: Thay vì khai báo 1 class với tên là DetailComponent cho module transaction thì hãy viết TransactionDetailComponent, hay function get(id:number) có thể thay thế thành getById(id:number)
- Phải phân biệt rõ ràng việc mình làm là reuse code hay mang cả thế giới bỏ vào 1 file, giữ cho mọi thứ thật đơn giản. Ví dụ: 
```typescript
function submit():void{
  if(this.form.invalid){
    return;
  }
  const data = this.form.getRawValue() as Payload
  this.api.create(data).subscribe(newItem => {
    this.router.navigate(['list']);
    this.toarst.open({type:'success',msg:'OK'})
  })
}
```
Ta có thể tách nhỏ function này để nó rõ nghĩa hơn như sau
```typescript
function navigateToList(url?:string):boolean{
  return this.router.navigate([url || 'list']);
}
function getPayload():Payload{
  return this.form.getRawValue()
}
function submit():void{
  if(this.form.invalid){
    return;
  }
  const data = this.getPayload();
  this.api.create(data).subscribe(newItem => {
    this.navigateToList();
    this.toarst.open({type:'success',msg:'OK'})
  })
}
```
- Sử dụng Generic Type cho abstract class, function, parameter, accesor ...
```typescript
@Component({
  selector: 'cms-user',
  templateUrl: './user.component.html',
  encapsulation: ViewEncapsulation.None,
  changeDetection: ChangeDetectionStrategy.OnPush,
  exportAs: 'cms-user',
})
export class CmsUserComponent implements OnInit{
  @Input()user:CmsUser;
  hasAvatar(user:APUser):boolean{
    return !!user.avatarUrl
  } 
}
```

```typescript
@Component({
  selector: 'ap-user',
  templateUrl: './user.component.html',
  encapsulation: ViewEncapsulation.None,
  changeDetection: ChangeDetectionStrategy.OnPush,
  exportAs: 'ap-user',
})
export class APUserComponent implements OnInit{
  @Input()user:APUser;
    hasAvatar(user:APUser):boolean{
    return !!user.profile.avatarUrl
  } 
}
```
ta có thể viết lại thành: 
```typescript

@Directive()
export abstract class UserBase<T> implements OnInit{
  @Input()user:T;
  abstract hasAvatar(user:T):boolean 
}
```

```typescript
@Component({
  selector: 'ap-user',
  templateUrl: './user.component.html',
  encapsulation: ViewEncapsulation.None,
  changeDetection: ChangeDetectionStrategy.OnPush,
  exportAs: 'ap-user',
})
export class APUserComponent<APUser> extends  BaseUser{
  override  hasAvatar(user:APUser):boolean{
    return !!user.profile.avatarUrl
  } 
}
```
- Code never lies, comments do. Code thì không bao giờ nói dối được, chỉ có comments. Nghĩa là không phải lúc nào comment cũng tốt. Nó chỉ tốt khi nó thật sự nói đúng ý của dòng code mà nó nói đến. Ví dụ:
```typescript
// check if it submit successfully, it navigate to list
 this.api.create(data).subscribe(newItem => {
    this.navigateToList();
    this.toarst.open({type:'success',msg:'OK'})
  })

 this.api.create(data).subscribe({
 next:(item => {
  // it navigate to list
    this.navigateToList();
    // notify
    this.toarst.open({type:'success',msg:'OK'})
   })
 
  })
```
- Memory leaks: Vấn đề này thường xuyên xảy ra với các application sử dung Rxjs. Khi chúng ta listen 1 event nhưng lại không báo cho nó biết khi nào thì nó phải dừng lại. Ví dụ:
```typescript
this.itemService.findItems()
  .pipe(
    map((items: Item[]) => items),
  ).subscribe()
```
có thể viết thành: 
```typescript
  private unsubscribe$: Subject<void> = new Subject<void>();
  ...
   this.itemService.findItems()
    .pipe(
       map(value => value.item)
       takeUntil(this._destroyed$)
     )
    .subscribe();
  ...
  public ngOnDestroy(): void {
    this.unsubscribe$.next();
    this.unsubscribe$.complete();
  }
```

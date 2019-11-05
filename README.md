# nagular-Interceptor
拦截器配置
import { Injectable } from '@angular/core';
import { HttpEvent, HttpInterceptor, HttpHandler, HttpRequest, HttpResponse } from '@angular/common/http';
import { Observable } from "rxjs/Observable";
import { ErrorObservable } from 'rxjs/observable/ErrorObservable';
import { catchError } from 'rxjs/operators';
import { mergeMap } from 'rxjs/operators';
import { MessageService } from 'abp-ng2-module/dist/src/message/message.service';
import { throwError as _observableThrow } from 'rxjs';
import { OpenIdConnectService } from '@shared/auth/openId-connect.service';
import * as _ from 'lodash';
@Injectable()
export class ErrorSettingService implements HttpInterceptor {
    responseBlob;
    constructor(private messageService: MessageService, private _OpenIdConnectService: OpenIdConnectService, private _AppAuth: AppAuthService,) { }
    intercept(request: HttpRequest<any>, next: HttpHandler): Observable<any> {
        let req = request;
        配置参数设置
        req = request.clone({
            setHeaders: {
                'Accept-Language': 'zh-Hans,zh;q=0.9,en;q=0.8'
            }
        });
        return next.handle(req).pipe(mergeMap((event: any) => {
            const err = event.body && event.body.err;
            this.responseBlob = event instanceof HttpResponse ? event.body : (<any>event).error instanceof Blob ? (<any>event).error : undefined;
            if (event instanceof HttpResponse && event.status !== 200) {
                let reader = new FileReader();
                reader.onload = readerEvent => {
                    this.messageService.error(JSON.parse(readerEvent.target['result']).error.message);
                };
                reader.readAsText(err);
                return ErrorObservable.create(event);
            }
            return Observable.create(observer => observer.next(event)); // 请求成功返回响应
        }),
            catchError((res: HttpResponse<any>) => {   // 请求失败处理
                switch (res.status) {
                    case 500:
                        let reader = new FileReader();
                        reader.onload = readerEvent => {
                            if (readerEvent.target['result']) {
                                let message = JSON.parse(readerEvent.target['result']).error.message;
                                this.messageService.error(message);
                            }
                        };
                        reader.readAsText(res['error']);
                        break;
                    case 401:
                    case 403:
                        ........
                        break;
                    default:
                        let statusText = this.statusText[res.status];
                        this.messageService.error(statusText);
                        break;
                }
                return ErrorObservable.create(event);
            }));
    }
}

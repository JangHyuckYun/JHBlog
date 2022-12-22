---
title: "NestJS 파일 업로드 설정"
date: "2022-11-23"
desc: "sodi (Let's share our diary lives)프로젝트의 백엔드 부분을 진행하며, 어려웠던 부분에 대해 적어봤습니다."
alt: "NestJS 파일 업로드 설정"
category: "JavaScript"
tags: ["JavaScript", "NestJS"]
thumbnail: ./images/nestJsFileUploadLogo.png
---

# 시작하기 전
대학 과제물로 진행하던 프로젝트 (sodi / [sodi gitub pages](https://github.com/JangHyuckYun/sodi)) 를 진행하면서
생각보다 많은 에러가 있었다. 그 중에서 초기에 겨우겨우 해결한 것 중 하나가 업로드 관련 부분이다.
적어두지 않으면, 나중에 똑같이 구글링을 계속 하며 찾아볼 거 같아 정리해두기로 했다.

# 1. package.json
먼저 NestJS에서 파일 업로드 기능을 사용하기 위해,  
1. package.json에 **"@nestjs/platform-express"**, **"@types/multer"** 모듈이 있어야 한다.
   1. > npm i @nestjs/platform-express  
      > npm i -D @types/multer
      
# 2. Module
모듈 데코레이션 안에서, import 배열 안에 **"MulterModule.registerAsync()"** 추가

```typescript
      MulterModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (config: ConfigService) => ({
        storage: diskStorage({
          destination: function (req, file, cb) {
            // 파일저장위치 + 년월 에다 업로드 파일을 저장한다.
            // 요 부분을 원하는 데로 바꾸면 된다.
            // config.get을 사용하여, .env 파일 안에 작성되어 있는 "ATTACH_SAVE_PATH_WINDOW" 변수를
            // 불러왔다. 
            const dest = `${config.get('ATTACH_SAVE_PATH_WINDOW')}/${format(
              new Date(),
              '{yyyy}{MM}',
            )}/`;
            
            //자신이 만들고자 하는 파일의 폴더 주소가 없을 경우 생성
            if (!fs.existsSync(dest)) {
              fs.mkdirSync(dest, { recursive: true });
            }

            cb(null, dest);
          },
          filename: (req, file, cb) => {
            // 업로드 후 저장되는 파일명을 랜덤하게 업로드 한다.(동일한 파일명을 업로드 됐을경우 오류방지)
            const randomName = Array(32)
              .fill(null)
              .map(() => Math.round(Math.random() * 16).toString(16))
              .join('');
            return cb(null, `${randomName}${extname(file.originalname)}`);
          },
        }),
      }),
  })
```
  
# 3. Controller

```typescript
  @Post('create')
  @UseInterceptors(FilesInterceptor('files'))
  @Bind(UploadedFiles())
  @UseGuards(JwtAuthGuard)
  createBoard(
    @UploadedFiles() files: Array<Express.Multer.File>,
    @Body() createBoardDto: CreateBoardDto,
    @Req() req,
  ) {
    createBoardDto.userId = Number(req?.user?.sub ?? -1);
    this.boardService.createBoard(createBoardDto, files);
  }
```

컨트롤러에서 **fileUpload** 관련하여 설정할 때 **@UseInterceptors(FilesInterceptor('files'))**,
**@Bind(UploadedFiles())** / 매개변수 안의 **@UploadedFiles() files** 부분이 중요하다.

## FilesInterceptor(fieldName, maxCount, Options)
이 함수는 총 3개의 인자를 갖는다.
1. fieldName: 필드 이름으로 HTTP요청 시 파일의 field명
2. maxCount: 업로드 시 허용할 수 있는 최대 파일 수
3. Options - MulterOptions타입

### - Options
기본적인 업로드 옵션은,
1. storage: 저장 방식 설정 ( 디스크, 메모리 )
2. fileFilter: 파일 유효성 체크
3. limits: 업로드 시 제한 설정
각 컨틀롤러 함수마다 Options을 지정하면 매우 길어질 수 있다. 이를 방지하고자 공용으로 설정한 부분이, 2번에 설정한 Module 부분이다.
설정하는 방법은 2번에 설명한 것을 참고하면 된다.
   1. ConfigService를 이용하여 Controller에서 설정하려는 경우 ( .env 값들을 Controller에서 사용하려는 경우 ), 컨트롤러및 해당 모듈에 추가 설정이 필요할 것이다.
   
> 파일들을 단일로 업로드 하는 경우,  
> @UseInterceptors(FilesInterceptor('files'))의 FilesInterceptor('files') 함수를 => FileInterceptor('file') 함수로,  
> @Bind(UploadedFiles())의 UploadedFiles() 함수를 => UploadedFile()함수로,  
> 매개변수의 @UploadedFiles() files,에서 @UploadedFiles() 데코레이터를 => @UploadedFile() file으로 변경해주면 된다.  


# 4. Check
데코레이터들을 통해 처리되기에, Controller타고 들어와서, 모듈에서 지정한 폴더를 직접 들어가보면 파일이 이미 생성 되있고,
console을 찍어보면 파일에 대한 정보가 출력될 것이다. 이 정보를 이용해 db에 넣거나, 개발자가 원하는대로 하면 된다.


# 마치며
nestJS를 공부하면서 설정을 하다보니, @UseInterceptors에서, @UploadedFiles() files의 콘솔을 찍으면 값이 안들어오거나,
files가 아닌 다른 dto에 파일 값이 들어가지는 등 갖가지 오류를 맛볼 수 있었다. ( 깊이 공부를 안한 탓이다.. )
그래도 직접 맛대며 하다 보니 이제는 대강 흐름을 알 수 있게 되었다.


# 참고자료
1. https://docs.nestjs.com/techniques/file-upload
2. https://any-ting.tistory.com/121
3. https://velog.io/@yiyb0603/Nest.js%EC%97%90%EC%84%9C-%ED%8C%8C%EC%9D%BC-%EC%97%85%EB%A1%9C%EB%93%9C%ED%95%98%EA%B8%B0
4. https://stove99.github.io/nodejs/2021/04/20/nestjs-file-upload/
5. https://any-ting.tistory.com/121

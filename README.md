## NestJS-Rest-API-with-MongoDB-CRUD


#### Project Create 
> powershell e daw
```bash
nest new nest-app
```
```bash
cd nest-app
```
---


#### install @nestjs/config
```bash
npm i @nestjs/config
```
---

>#### Project Root e .env file banaw.
---


>#### Note: browse- https://www.mongodb.com/cloud/atlas/register account create koro, clusters create koro
>#### .env te database connect koro. [MongoDB Atlas Setup](https://github.com/Omarmdwasimuddin/mongodb-atlas)
---


#### `example.env`
```bash
MONGODB_USERNAME=""
MONGODB_PASSWORD=""
MONGODB_URI=""
```
---


#### Install mongoose
```bash
npm i @nestjs/mongoose mongoose
```
---


>##### Note: app.module.ts file e add koro- ConfigModule.forRoot(),
>##### MongooseModule.forRoot(process.env.MONGODB_URI!)
##
#### `app.module.ts`
```bash
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from '@nestjs/config';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [ ConfigModule.forRoot(), MongooseModule.forRoot(process.env.MONGODB_URI!) ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
---










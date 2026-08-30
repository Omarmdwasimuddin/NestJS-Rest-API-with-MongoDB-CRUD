# NestJS REST API with MongoDB CRUD 


এখানে দেখানো হবে কীভাবে NestJS দিয়ে MongoDB-এর সাথে connect করে একটা পুরোপুরি কাজ করা **CRUD REST API** (Create, Read, Update, Delete) বানানো যায় — `students` নামের একটা resource দিয়ে উদাহরণ হিসেবে।

---

## ধাপ ১: Project তৈরি করা

PowerShell-এ (বা যেকোনো terminal-এ) দাও:

```bash
nest new nest-app
```

```bash
cd nest-app
```

---

## ধাপ ২: `@nestjs/config` Install করা

Environment variable (যেমন `.env` file) ব্যবহার করতে হলে `@nestjs/config` package লাগবে:

```bash
npm i @nestjs/config
```

---

## ধাপ ৩: `.env` File বানানো

Project-এর root-এ একটা `.env` file বানাও।

> **নোট:** [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register)-এ গিয়ে একটা account বানাও, cluster তৈরি করো, এরপর সেই cluster থেকে connection string নিয়ে `.env`-এ database connect করো। বিস্তারিত ধাপে ধাপে setup দেখতে চাইলে এই গাইড দেখো: [MongoDB Atlas Setup](https://github.com/Omarmdwasimuddin/mongodb-atlas)।

### `example.env`

```bash
MONGODB_USERNAME=""
MONGODB_PASSWORD=""
MONGODB_URI=""
```

---

## ধাপ ৪: Mongoose Install করা

```bash
npm i @nestjs/mongoose mongoose
```

---

## ধাপ ৫: `app.module.ts` Setup করা

> **নোট:** `app.module.ts`-এ যোগ করতে হবে:
> - `ConfigModule.forRoot()`
> - `MongooseModule.forRoot(process.env.MONGODB_URI!)`

### `app.module.ts`

```ts
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

## ধাপ ৬: Module, Service, Controller তৈরি করা

```bash
nest g module students
```

```bash
nest g service students
```

```bash
nest g controller students
```

এই তিনটা command চালালে `students` নামের একটা folder তৈরি হবে, যার ভিতরে module, service, আর controller file automatic জেনারেট হয়ে যাবে।

---

## ধাপ ৭: Schema File তৈরি করা

`students.schema.ts` নামের একটা file manually বানাতে হবে — CLI দিয়ে এটা automatic জেনারেট হয় না।

![students.schema.ts ফাইল](https://github.com/user-attachments/assets/224defc3-7994-46b4-b308-f5bb114e320d)

### `students.schema.ts`

```ts
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { Document } from "mongoose";

export type StudentDocument = Student & Document;

@Schema( { timestamps: true } )
export class Student {
    @Prop( { required: true } )
    name!: string;
    
    @Prop( { required: true } )
    age!: number;

    @Prop()
    email?: string;
}

export const StudentSchema = SchemaFactory.createForClass( Student );
```

**এখানে কী হচ্ছে:**
- `@Schema({ timestamps: true })` — `timestamps: true` দিলে Mongoose automatic `createdAt` আর `updatedAt` field যোগ করে দেয়
- `name` আর `age` — `required: true` দেওয়া, মানে এই দুইটা field ছাড়া document save হবে না
- `email` — optional (`?` চিহ্ন দিয়ে বোঝানো হয়েছে, `required` দেওয়া হয়নি)
- `StudentDocument` type — `Student` class আর Mongoose-এর `Document` — দুটো মিলিয়ে বানানো হয়েছে, যাতে service-এ পুরো Mongoose document-এর সব method (`.save()`, `._id` ইত্যাদি) পাওয়া যায়

---

## ধাপ ৮: Module Setup করা

### `students.module.ts`

```ts
import { Module } from '@nestjs/common';
import { StudentsService } from './students.service';
import { StudentsController } from './students.controller';
import { MongooseModule } from '@nestjs/mongoose';
import { Student, StudentSchema } from './students.schema';

@Module({
  imports: [ MongooseModule.forFeature([{ name: Student.name, schema: StudentSchema }]) ],
  providers: [StudentsService],
  controllers: [StudentsController]
})
export class StudentsModule {}
```

এখানে `MongooseModule.forFeature()` দিয়ে `Student` model-টা এই module-এর scope-এ register করা হয়েছে — এর ফলে `StudentsService`-এ `@InjectModel()` দিয়ে এই model inject করা সম্ভব হবে।

---

## ধাপ ৯: Service লেখা (Business Logic)

### `students.service.ts`

```ts
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Student, StudentDocument } from './students.schema';
import { Model } from 'mongoose';

@Injectable()
export class StudentsService {
    constructor(
        @InjectModel(Student.name) private studentModel: Model<StudentDocument>
    ){}

    async createStudent(data: Partial<Student>): Promise<Student>{
        const newStudent = new this.studentModel(data);
        return newStudent.save();
    }

    async getAllStudent(): Promise<Student[]>{
        return this.studentModel.find().exec();
    }

    async getStudentById(id: string): Promise<Student | null>{
        return this.studentModel.findById(id).exec();
    }

    async upateStudent(id: string, data: Partial<Student>): Promise<Student | null>{
        //return this.studentModel.findByIdAndUpdate(id, data, { new: true }).exec();
        const update = await this.studentModel.findByIdAndUpdate(id, {
            name: data.name ?? null,
            age: data.age ?? null,
            email: data.email ?? null,
        }, { overwrite: true, new: true });
        return update;
    }

    async patchStudent(id: string, data: Partial<Student>): Promise<Student | null>{
        return this.studentModel.findByIdAndUpdate(id, { $set: data }, { new: true }).exec();
    }

    async deleteStudent(id: string): Promise<Student | null>{
        return this.studentModel.findByIdAndDelete(id).exec();
    }
}
```

### প্রতিটা method কী করছে

| Method | কাজ |
|---|---|
| `createStudent()` | নতুন `student` document বানিয়ে database-এ save করে |
| `getAllStudent()` | সবগুলো student একসাথে fetch করে |
| `getStudentById()` | নির্দিষ্ট `id` দিয়ে একজন student খুঁজে বের করে |
| `upateStudent()` | **PUT-স্টাইল update** — পুরো document পুরোপুরি নতুন data দিয়ে replace করে (`overwrite: true`); কোনো field না দিলে সেটা `null` হয়ে যাবে |
| `patchStudent()` | **PATCH-স্টাইল update** — `$set` দিয়ে শুধু যেসব field পাঠানো হয়েছে সেগুলোই update করে, বাকিগুলো অপরিবর্তিত থাকে |
| `deleteStudent()` | নির্দিষ্ট `id`-এর student মুছে ফেলে |

> **PUT vs PATCH-এর পার্থক্য মনে রাখার সহজ উপায়:** PUT মানে পুরো object replace, PATCH মানে শুধু যা পাঠানো হয়েছে তা আপডেট। এই জন্যই `upateStudent()`-এ `overwrite: true` ব্যবহার করা হয়েছে (পুরো document বদলে যায়), আর `patchStudent()`-এ `$set` ব্যবহার করা হয়েছে (নির্দিষ্ট field-ই বদলায়)।

---

## ধাপ ১০: Controller লেখা (Route Handler)

### `students.controller.ts`

```ts
import { Body, Controller, Delete, Get, Param, Patch, Post, Put } from '@nestjs/common';
import { StudentsService } from './students.service'
import { Student } from './students.schema';

@Controller('students')
export class StudentsController {
    constructor(private readonly studentService: StudentsService){}

    @Post()
    async createStudent(@Body() data: Partial<Student>){
        return this.studentService.createStudent(data);
    }

    @Get()
    async getAllStudent(){
        return this.studentService.getAllStudent();
    }

    @Get(':id')
    async getStudentById(@Param('id') id: string){
        return this.studentService.getStudentById(id);
    }

    @Put(':id')
    async updateStudent(@Param('id') id: string, @Body() data: Partial<Student>){
        return this.studentService.upateStudent(id, data);
    }

    @Patch(':id')
    async patchStudent(@Param('id') id: string, @Body() data: Partial<Student> ){
        return this.studentService.patchStudent(id, data);
    }

    @Delete(':id')
    async deleteStudent(@Param('id') id: string){
        return this.studentService.deleteStudent(id);
    }

}
```

### Route-গুলো একনজরে

| HTTP Method | Route | কাজ |
|---|---|---|
| `POST` | `/students` | নতুন student তৈরি |
| `GET` | `/students` | সবগুলো student দেখা |
| `GET` | `/students/:id` | নির্দিষ্ট student দেখা |
| `PUT` | `/students/:id` | পুরো student document replace |
| `PATCH` | `/students/:id` | নির্দিষ্ট field আপডেট |
| `DELETE` | `/students/:id` | student মুছে ফেলা |

`@Controller('students')` দিয়ে বলে দেওয়া হয়েছে এই controller-এর সবগুলো route `/students` prefix দিয়ে শুরু হবে। `@Body()` দিয়ে request body থেকে data নেওয়া হয়, `@Param('id')` দিয়ে URL-এর `:id` অংশটা নেওয়া হয়।

---

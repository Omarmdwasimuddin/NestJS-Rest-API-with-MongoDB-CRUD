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


#### Create module, service, controller
```bash
nest g module students
```
```bash
nest g service students
```
```bash
nest g controller students
```
---


>#### manualy file banao students.schema.ts
<img width="246" height="67" alt="image" src="https://github.com/user-attachments/assets/224defc3-7994-46b4-b308-f5bb114e320d" />

##

#### `students.schema.ts`
```bash
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
---



#### `students.module.ts`
```bash
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
---



#### `students.service.ts`
```bash
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Student, StudentDocument } from './students.schema';
import { Model } from 'mongoose';

@Injectable()
export class StudentsService {
    constructor(
        @InjectModel(Student.name) private studentModel: Model<StudentDocument>
    ) {}

    async createStudent(data: Partial<Student>): Promise<Student> {
        const newStudent = new this.studentModel(data);
        return newStudent.save();
    }

    async getAllStudents(): Promise<Student[]> {
        return this.studentModel.find().exec();
    }

    async getStudentById(id: string): Promise<Student | null> {
        return this.studentModel.findById(id).exec();
    }

    async updateStudent(id: string, data: Partial<Student>): Promise<Student | null> {
        return this.studentModel.findByIdAndUpdate(id, data, { new: true }).exec();
    }

}
```
---


#### `students.controller.ts`
```bash
import { Body, Controller, Get, Param, Post, Put } from '@nestjs/common';
import { StudentsService } from './students.service';
import { Student } from './students.schema';

@Controller('students')
export class StudentsController {
    constructor(private readonly studentsService: StudentsService) {}

    @Post()
    async createStudent(@Body() data: Partial<Student>) {
        return this.studentsService.createStudent(data);
    }

    @Get()
    async getAllStudents() {
        return this.studentsService.getAllStudents();
    }

    @Get(':id')
    async getStudentById(@Param('id') id: string) {
        return this.studentsService.getStudentById(id);
    }

    @Put(':id')
    async updateStudent(@Param('id') id: string, @Body() data: Partial<Student>) {
        return this.studentsService.updateStudent(id, data);
    }

}
```
---

# database-prisma
// src/exercise-match/exercise-match.module.ts
import { Module } from '@nestjs/common';
import { ExerciseMatchService } from './exercise-match.service';
import { ExerciseMatchController } from './exercise-match.controller';
import { PrismaService } from '../prisma/prisma.service';

@Module({
  controllers: [ExerciseMatchController],
  providers: [ExerciseMatchService, PrismaService],
})
export class ExerciseMatchModule {}

// src/exercise-match/exercise-match.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { ExerciseMatch, Prisma } from '@prisma/client';

@Injectable()
export class ExerciseMatchService {
  constructor(private prisma: PrismaService) {}

  async createExerciseMatch(data: Prisma.ExerciseMatchCreateInput): Promise<ExerciseMatch> {
    return this.prisma.exerciseMatch.create({
      data,
    });
  }

  async exerciseMatches(params: {
    skip?: number;
    take?: number;
    cursor?: Prisma.ExerciseMatchWhereUniqueInput;
    where?: Prisma.ExerciseMatchWhereInput;
    orderBy?: Prisma.ExerciseMatchOrderByWithRelationInput;
  }): Promise<ExerciseMatch[]> {
    const { skip, take, cursor, where, orderBy } = params;
    return this.prisma.exerciseMatch.findMany({
      skip,
      take,
      cursor,
      where,
      orderBy,
    });
  }
}

// src/exercise-match/exercise-match.controller.ts
import { Controller, Get, Post, Body, Query, ValidationPipe } from '@nestjs/common';
import { ExerciseMatchService } from './exercise-match.service';
import { ExerciseMatch } from '@prisma/client';
import { IsInt, IsString, IsOptional, Min, Max } from 'class-validator';

class CreateExerciseMatchDto {
  @IsInt()
  expertId: number;

  @IsInt()
  clientId: number;

  @IsString()
  specialization: string;

  @IsString()
  availability: string;

  @IsInt()
  @Min(1)
  @Max(5)
  rating: number;
}

class GetExerciseMatchesDto {
  @IsOptional()
  @IsString()
  specialization?: string;

  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(5)
  rating?: number;
}

@Controller('exercise-match')
export class ExerciseMatchController {
  constructor(private readonly exerciseMatchService: ExerciseMatchService) {}

  @Post()
  async create(@Body(new ValidationPipe()) createExerciseMatchDto: CreateExerciseMatchDto): Promise<ExerciseMatch> {
    return this.exerciseMatchService.createExerciseMatch(createExerciseMatchDto);
  }

  @Get()
  async findAll(@Query(new ValidationPipe()) query: GetExerciseMatchesDto): Promise<ExerciseMatch[]> {
    const where = {};
    if (query.specialization) {
      where['specialization'] = query.specialization;
    }
    if (query.rating) {
      where['rating'] = query.rating;
    }

    return this.exerciseMatchService.exerciseMatches({ where });
  }
}

// src/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model ExerciseMatch {
  id             Int      @id @default(autoincrement())
  expertId       Int
  clientId       Int
  specialization String
  availability   String
  rating         Int
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
}

// src/prisma/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}

// src/app.module.ts
import { Module } from '@nestjs/common';
import { ExerciseMatchModule } from './exercise-match/exercise-match.module';
import { PrismaModule } from './prisma/prisma.module';

@Module({
  imports: [ExerciseMatchModule, PrismaModule],
  controllers: [],
  providers: [],
})
export class AppModule {}

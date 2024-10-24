# Api Auth Nestjs template

This repository provides a base template to develop APIS in Nestjs with the authentication and authorization functions with user roles already implemented. Designed to be reusable in future projects, this template offers a robust structure that facilitates the rapid creation of safe and scalable applications.

## Getting started

### Requirements

Before starting, make sure you have at least those components on your workstation:

- [NodeJS 20.x](https://nodejs.org/en/download/package-manager)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Nest CLI](https://docs.nestjs.com/cli/overview#installation) command line interface tool

```sh
# windows
npm install -g @nestjs/cli
# Linux and Mac
sudo npm install -g @nestjs/cli
```

- [Yarn](https://classic.yarnpkg.com/lang/en/docs/install/)

```sh
# windows
npm install --global yarn
# Linux and Mac
sudo npm install --global yarn
```

> [!NOTE]
> For a correct installation of the Nest Cli and yarn in Windows it is necessary to open the terminal as administrator and in Linux and Mac use the sudo command `sudo npm install -g @nestjs/cli`, `sudo npm install --global yarn`

### Project setup

1. Start cloning this repository.

```sh
git clone https://github.com/ayunierto/api-auth-nestjs-base.git my-project
```

2. The next thing will be to install all the dependencies of the project.

```sh
cd ./my-project
yarn
```

3. For this application to work correctly, it is necessary to configure the following environment variables, create a new `.env` file.

```sh
cp .env.example .env
```

- **`DB_PASSWORD`**: The password used to connect to the database. Be sure to use a safe password.  
  _Example:_ `MyPassword`

- **`DB_NAME`**: The name of the database that the application will use to store and recover information.  
  _Example:_ `dbname`

- **`DB_HOST`**: The server address where the database is housed. For local development environments, it can be `localhost`.  
  _Example:_ `localhost`

- **`DB_PORT`**: The port through which the application will connect to the database. The default value for postgresql is `5432`.  
  _Example:_ `5432`

- **`DB_USERNAME`**: The username of the database. Generally, this will be the main user to access the database.  
  _Example:_ `postgres`

- **`JWT_SECRET`**: The secret key used to sign and verify the tokens JWT (JSON Web tokens) in the authentication process. It is important that this key is complex and remains safe.  
  _Example:_ `thisismysecretpasswordforjwt`

### Create and start database container

```sh
docker-compose up -d
```

### Compile and run the project

```bash
# development
yarn run start

# development watch mode
yarn run start:dev

# production mode
yarn run start:prod
```

## Use

1. Generate a new CRUD resource

```sh
# The flag --no-spec omits the creation of tests
nest g res products --no-spec
```

> [!NOTE]
> Change `products` by the name of your resource.

2. Add `@Entity()` decorator to products.entity.ts

```ts
import { Entity } from 'typeorm'; // <- Add this line

@Entity() // <- Add this line
export class Products {}
```

3. Add columns in `product.entity.ts`

```ts
import { Column, Entity, PrimaryGeneratedColumn } from 'typeorm'; // <- Update dependencies

@Entity()
export class Product {
  // PrimaryKey (using uuid)
  @PrimaryGeneratedColumn('uuid')
  id: string;

  //  Add text type title column and define that you have the unique value
  @Column('text', {
    unique: true,
  })
  title: string;

  @Column('text')
  description?: string; // <- Optional (?)

  // Add more columns
  // ...
}
```

4. Import TypeOrmModule in `products.module.ts`

```ts
import { Module } from '@nestjs/common';
import { ProductsService } from './products.service';
import { ProductsController } from './products.controller';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Product } from './entities/product.entity'; // <- Add this line

@Module({
  controllers: [ProductsController],
  providers: [ProductsService],
  imports: [TypeOrmModule.forFeature([Product])], // <- Add this line
})
export class ProductsModule {}
```

5. Add validations in our `create-product.dto.ts`

```ts
import { IsOptional, IsString, MinLength } from 'class-validator';

export class CreateProductDto {
  @IsString()
  @MinLength(3)
  title: string;

  @IsString()
  @IsOptional()
  description?: string;
}
```

6. Update controller `products.controller.ts`

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  ParseUUIDPipe,
} from '@nestjs/common';
import { ProductsService } from './products.service';
import { CreateProductDto } from './dto/create-product.dto';
import { UpdateProductDto } from './dto/update-product.dto';

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Post()
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  @Get()
  findAll() {
    return this.productsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    return this.productsService.findOne(id);
  }

  @Patch(':id')
  update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    return this.productsService.update(id, updateProductDto);
  }

  @Delete(':id')
  remove(@Param('id', ParseUUIDPipe) id: string) {
    return this.productsService.remove(id);
  }
}
```

<!-- todo -->

7. Update service `products.service.ts`

```ts
import {
  BadRequestException,
  Injectable,
  InternalServerErrorException,
  Logger,
  NotFoundException,
  Query,
} from '@nestjs/common';
import { CreateProductDto } from './dto/create-product.dto';
import { UpdateProductDto } from './dto/update-product.dto';
import { Repository } from 'typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Product } from './entities/product.entity';

@Injectable()
export class ProductsService {
  private readonly logger = new Logger('ProductsService');

  constructor(
    @InjectRepository(Product)
    private readonly productRepository: Repository<Product>,
  ) {}

  async create(createProductDto: CreateProductDto) {
    try {
      const product = this.productRepository.create(createProductDto);
      await this.productRepository.save(product);
      return product;
    } catch (error) {
      this.handleDBExceptions(error);
    }
  }

  async findAll() {
    return await this.productRepository.find();
  }

  async findOne(id: string) {
    const product = await this.productRepository.findOneBy({ id });
    if (!product) throw new NotFoundException();

    return product;
  }

  async update(id: string, updateProductDto: UpdateProductDto) {
    const product = await this.productRepository.preload({
      id,
      ...updateProductDto,
    });
    if (!product) throw new NotFoundException();
    try {
      await this.productRepository.save(product);
      return product;
    } catch (error) {
      this.handleDBExceptions(error);
    }
  }

  async remove(id: string) {
    const product = await this.findOne(id);
    await this.productRepository.remove(product);
    return product;
  }

  // Handle Errors
  private handleDBExceptions(error: any) {
    if (error.code === '23505') throw new BadRequestException(error.detail);

    this.logger.error(error);
    throw new InternalServerErrorException(
      'Unexpected error, check server logs.',
    );
  }
}
```

8. Generate a new module for Pagination (Optional)

```sh
nest g module common
```

8.1 Create dto in `src/common/dto/pagination.dto.ts`

```ts
// pagination.dto.ts
import { IsOptional, IsPositive } from 'class-validator';

export class PaginationDto {
  @IsOptional()
  @IsPositive()
  @Type(() => Number)
  limit?: number;

  @IsOptional()
  @Min(0)
  @Type(() => Number)
  offset?: number;
}
```

8.2 Modify controller `products.controller.ts`

```ts
// ...
import { PaginationDto } from 'src/common/dto/pagination.dto';
// ...

@Controller('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  // ...

  // Add decorator @Query and PaginationDto to method findAll()
  @Get()
  findAll(@Query() paginationDto: PaginationDto) {
    return this.productsService.findAll(paginationDto);
  }

  // ...
}
```

8.3 Modify service `products.service.ts`

```ts
// ...
import { PaginationDto } from 'src/common/dto/pagination.dto';
// ...

@Injectable()
export class ProductsService {
  private readonly logger = new Logger('ProductsService');

  // ...

  async findAll(paginationDto: PaginationDto) {
    const { limit = 10, offset = 0 } = paginationDto;
    return await this.productRepository.find({
      take: limit,
      skip: offset,
    });
  }

// ...


```

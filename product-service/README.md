# Product Service
This service focus on product management. This implement DTO, Model and business logic
## Database
We use MySQL for Product Service. To prevent init many connections to database, we implement Database Singleton
```
export declare class Database extends SqlConnection {
    private static instance;
    private bookshelf;
    private constructor();
    static getInstance(): Database;
    getBookshelf(): Bookshelf;
}
```
## Migrations
Because we use RDMS, we need to keep schema consistent by using migrations
To generate new migration:
```
npm run migrate:make {filename}
```
## Dto
To transfer object between internal components and response to client
## Repositoy pattern
We using repository pattern to encapsulate the logic required to access data sources. They centralize common data access functionality, providing better maintainability and decoupling the infrastructure or technology used to access databases from the domain model layer.
![alt text](https://drive.google.com/file/d/1jxnUcHL_96NL50rmFxzZNrMNfZRzE-G8/view?usp=sharing)
### Repository
```
export declare class BaseRepository<T extends BaseModel<T>, X extends BaseDto> {
    protected model: {
        new (attributes?: any, isNew?: boolean): T;
    };
    protected dto: {
        new (): X;
    };
    protected converter: {
        toModel: (data: X) => any;
        fromModel: (data: any, filters: string[]) => X;
    };
    constructor(model: {
        new (attributes?: any, isNew?: boolean): T;
    }, dto: {
        new (): X;
    }, converter: {
        toModel: (data: X) => any;
        fromModel: (data: any, filters: string[]) => X;
    });
    update(data: X, t?: Transaction): Promise<X>;
    insert(data: X, t?: Transaction): Promise<X>;
    delete(id: string, t?: Transaction): Promise<void>;
    updateByQuery(callback: (qb: QueryBuilder) => void, data: any, t?: Transaction): Promise<T>;
    findAll(ids: string[], related?: string[], filters?: string[]): Promise<X[]>;
    findOne(id: string, related?: string[], filters?: string[]): Promise<X>;
    countByQuery(callback: (qb: QueryBuilder) => void): Promise<number>;
    countFetchByQuery(callback: (qb: QueryBuilder) => void): Promise<number>;
    findByQuery(callback: (qb: QueryBuilder) => void, related?: string[], filters?: string[]): Promise<X[]>;
    findOneByQuery(callback: (qb: QueryBuilder) => void, related?: string[], filters?: string[]): Promise<X>;
    countAndQuery(countQuery: (qb: QueryBuilder) => void, findQuery: (qb: QueryBuilder) => void, related?: string[], filters?: string[]): Promise<CollectionWrap<X>>;
}
```
### Models
This class extends bookshelf model for ORM using.
```
export declare abstract class BaseModel<T> extends BaseModel_base<any> {
    private static generateUuid;
    constructor(attributes?: any, isNew?: boolean);
    get idProperty(): string;
    get isDelete(): string;
    get hasTimestamps(): string[];
    initialize(): void;
}
```
### Services
Encapsulate the business logic. It is "Client Business Logic" in image above
## Activity Middleware
Init AMQP connection and publish message to activity topic
## Routers
Define routing and handling
## Dtos
### Product Dto
```
export declare class ProductDto extends BaseDto {
    name: string;
    price: number;
    brand?: string;
    color?: string;
    description?: string;
    static fromModel(model: ProductModel, filters: string[]): ProductDto;
    static toModel(dto: ProductDto): ProductModel;
    static fromRequest(body: any): ProductDto;
    validate(): void;
}
```
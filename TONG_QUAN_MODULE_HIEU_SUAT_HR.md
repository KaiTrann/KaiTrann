# Tong Quan Module Hieu Suat & Phan Tich Nhan Su

File nay tom tat phan code tuong ung voi nhanh trong so do:

- Hieu suat & Phan tich nhan su
- KPI / OKR
- Danh gia hieu suat
- Phan tich du lieu HR
- Dashboard quan tri

Project duoc tach thanh 3 lop chinh:

- Frontend: React, nam trong `frontend/src`
- Backend: NestJS + TypeORM, nam trong `backend/src`
- Database: MySQL schema/data, nam trong `payroll_2026.sql`

## 1. Tong Quan Vi Tri Code

Phan nay khong nam trong mot thu muc rieng ten "Hieu suat & Phan tich nhan su". Code duoc chia theo tung lop:

| Thanh phan | Vi tri |
| --- | --- |
| Menu dieu huong | `frontend/src/components/layout/Navbar.jsx` |
| Khai bao route | `frontend/src/routes/AppRoutes.jsx` |
| Cau hinh form/table KPI va danh gia | `frontend/src/pages/managementConfigs.js` |
| Trang CRUD dung chung | `frontend/src/pages/ManagementPage.jsx` |
| API service frontend | `frontend/src/api/services/srsService.js` |
| Dashboard quan tri | `frontend/src/pages/Dashboard.jsx` |
| Bao cao/phan tich HR | `frontend/src/pages/Reports.jsx` |
| Ham xu ly analytics | `frontend/src/utils/analytics.js` |
| Backend controllers | `backend/src/modules/srs-features/srs.controllers.ts` |
| Backend services | `backend/src/modules/srs-features/srs.services.ts` |
| Module NestJS dang ky controller/service/entity | `backend/src/modules/srs-features/srs-features.module.ts` |
| CRUD service dung chung | `backend/src/modules/crud/crud.service.ts` |
| Entity KPI/OKR | `backend/src/database/payroll/entities/kpi-okr.entity.ts` |
| Entity danh gia hieu suat | `backend/src/database/payroll/entities/performance-review.entity.ts` |
| Bang database KPI | `payroll_2026.sql`, bang `kpi_okr` |
| Bang database danh gia | `payroll_2026.sql`, bang `performance_reviews` |

## 2. KPI / OKR

### Frontend

Route:

```jsx
// frontend/src/routes/AppRoutes.jsx
<Route
  path="/kpi-okr"
  element={<AuthorizedPage path="/kpi-okr"><ManagementPage config={managementConfigs.kpiOkr} /></AuthorizedPage>}
/>
```

Menu:

```jsx
// frontend/src/components/layout/Navbar.jsx
{
  key: 'performance',
  label: 'Hieu suat',
  children: [
    { to: '/kpi-okr', label: 'KPI / OKR', desc: 'Chi so hieu suat muc tieu', icon: Star },
  ],
}
```

Cau hinh form/table:

```js
// frontend/src/pages/managementConfigs.js
kpiOkr: {
  endpoint: '/kpi-okr',
  idField: 'KpiID',
  title: 'KPI / OKR',
  fields: [
    { name: 'KpiID', label: 'Ma KPI', readOnly: true },
    { name: 'EmployeeID', label: 'Ma nhan vien', type: 'number' },
    { name: 'Period', label: 'Chu ky' },
    { name: 'Title', label: 'Muc tieu' },
    { name: 'TargetValue', label: 'Chi tieu', type: 'number' },
    { name: 'ActualValue', label: 'Thuc dat', type: 'number' },
    { name: 'Weight', label: 'Trong so (%)', type: 'number' },
    { name: 'Score', label: 'Diem', type: 'number' },
    { name: 'Status', label: 'Trang thai', options: statusOptions, badge: true },
  ],
}
```

`ManagementPage` se doc config nay de tu dong tao:

- Bang danh sach KPI
- O tim kiem
- Nut them moi
- Modal them/sua/xoa
- Goi API list/create/update/delete

### Backend

Controller:

```ts
// backend/src/modules/srs-features/srs.controllers.ts
@Controller('kpi-okr')
@Permissions('kpi.manage')
export class KpiOkrController extends BaseCrudController {
  constructor(service: KpiOkrService) {
    super(service);
  }
}
```

Service:

```ts
// backend/src/modules/srs-features/srs.services.ts
@Injectable()
export class KpiOkrService extends CrudService<KpiOkr & Record<string, unknown>> {
  constructor(@InjectRepository(KpiOkr, 'payrollConnection') repo: Repository<KpiOkr>, audit: AuditService) {
    super(repo as never, {
      entityName: 'KpiOkr',
      idField: 'KpiID',
      searchFields: ['Period', 'PeriodType', 'Title', 'Description', 'Status'],
      softDeleteField: 'Status',
      softDeleteValue: 'Cancelled',
      defaultOrder: { KpiID: 'DESC' },
    }, audit);
  }
}
```

KPI/OKR dung CRUD chung, nghia la service khong tu viet lai find/create/update/delete ma ke thua tu `CrudService`.

### Database Entity

```ts
// backend/src/database/payroll/entities/kpi-okr.entity.ts
@Entity('kpi_okr')
export class KpiOkr {
  @PrimaryGeneratedColumn()
  KpiID: number;

  @Column()
  EmployeeID: number;

  @Column({ type: 'varchar', length: 20 })
  Period: string;

  @Column({ type: 'varchar', length: 10, default: 'Quarterly' })
  PeriodType: string;

  @Column({ type: 'varchar', length: 255 })
  Title: string;

  @Column({ type: 'decimal', precision: 10, scale: 2, nullable: true })
  TargetValue: number;

  @Column({ type: 'decimal', precision: 10, scale: 2, nullable: true })
  ActualValue: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, default: 100 })
  Weight: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Score: number;

  @Column({ type: 'decimal', precision: 15, scale: 2, nullable: true, default: 0 })
  BonusAmount: number;

  @Column({ type: 'varchar', length: 20, default: 'Active' })
  Status: string;
}
```

Y nghia cac cot chinh:

| Cot | Y nghia |
| --- | --- |
| `KpiID` | Ma KPI tu tang |
| `EmployeeID` | Nhan vien duoc gan KPI |
| `Period` | Ky KPI, vi du `2026-Q2` hoac `2026-05` |
| `PeriodType` | Loai ky: Monthly, Quarterly... |
| `Title` | Ten muc tieu |
| `TargetValue` | Chi tieu can dat |
| `ActualValue` | Ket qua thuc te |
| `Weight` | Trong so KPI |
| `Score` | Diem KPI |
| `BonusAmount` | Tien thuong KPI dung khi tinh luong |
| `Status` | Trang thai: Active, Approved, Cancelled... |

## 3. Danh Gia Hieu Suat

### Frontend

Route:

```jsx
// frontend/src/routes/AppRoutes.jsx
<Route
  path="/performance-evaluation"
  element={<AuthorizedPage path="/performance-evaluation"><ManagementPage config={managementConfigs.performanceEvaluation} /></AuthorizedPage>}
/>
```

Cau hinh:

```js
// frontend/src/pages/managementConfigs.js
performanceEvaluation: {
  endpoint: '/performance-evaluation',
  idField: 'ReviewID',
  title: 'Danh gia hieu suat',
  fields: [
    { name: 'ReviewID', label: 'Ma danh gia', readOnly: true },
    { name: 'EmployeeID', label: 'Ma nhan vien', type: 'number' },
    { name: 'ReviewPeriod', label: 'Ky danh gia' },
    { name: 'ReviewDate', label: 'Ngay danh gia', type: 'date' },
    { name: 'ReviewerID', label: 'Nguoi danh gia', type: 'number' },
    { name: 'OverallScore', label: 'Diem tong', type: 'number' },
    { name: 'Grade', label: 'Xep loai' },
    { name: 'Status', label: 'Trang thai', options: statusOptions, badge: true },
  ],
}
```

Trang nay cung dung `ManagementPage`, nen co chuc nang:

- Xem danh sach phieu danh gia
- Them phieu danh gia
- Sua phieu danh gia
- Xoa/vo hieu hoa phieu danh gia
- Tim kiem theo ky, xep loai, trang thai, diem manh, muc tieu

### Backend Controller

```ts
// backend/src/modules/srs-features/srs.controllers.ts
@Controller('performance-evaluation')
@Permissions('performance.manage')
export class PerformanceEvaluationController extends BaseCrudController {
  constructor(private readonly performanceService: PerformanceEvaluationService) {
    super(performanceService);
  }

  @Post('auto-calculate')
  autoCalculate(@Body() body) {
    return this.performanceService.autoCalculate(body);
  }
}
```

Controller nay co 2 nhom chuc nang:

- CRUD chung: `GET`, `POST`, `PATCH`, `DELETE`
- Tinh tu dong: `POST /performance-evaluation/auto-calculate`

### Backend Service

Service co logic quan trong trong ham `autoCalculate`.

```ts
const attendanceScore = await this.calculateAttendanceScore(body.EmployeeID, period);
const kpiScore = await this.calculateKpiScore(body.EmployeeID, body.ReviewPeriod, period);
const overallScore = Number((kpiScore * 0.6 + attendanceScore * 0.4).toFixed(2));
```

Cong thuc:

```text
Diem tong = Diem KPI * 60% + Diem cham cong * 40%
```

Sau do service gan vao record:

```ts
record.OverallScore = overallScore;
record.Competency = this.toFivePoint(kpiScore);
record.Attitude = this.toFivePoint(attendanceScore);
record.Teamwork = this.toFivePoint(overallScore);
record.Productivity = this.toFivePoint(overallScore);
record.Grade = this.gradeFromScore(overallScore);
record.Status = body.Status ?? 'Submitted';
```

Ham `toFivePoint` doi diem 100 sang thang 5:

```ts
private toFivePoint(score: number) {
  return Number((Math.max(0, Math.min(100, score)) / 20).toFixed(2));
}
```

Ham xep loai:

```ts
private gradeFromScore(score: number) {
  if (score >= 95) return 'A+';
  if (score >= 85) return 'A';
  if (score >= 75) return 'B+';
  if (score >= 65) return 'B';
  if (score >= 50) return 'C';
  return 'D';
}
```

### Database Entity

```ts
// backend/src/database/payroll/entities/performance-review.entity.ts
@Entity('performance_reviews')
export class PerformanceReview {
  @PrimaryGeneratedColumn()
  ReviewID: number;

  @Column()
  EmployeeID: number;

  @Column({ type: 'varchar', length: 20 })
  ReviewPeriod: string;

  @Column({ type: 'date', nullable: true })
  ReviewDate: Date;

  @Column({ nullable: true })
  ReviewerID: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  OverallScore: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Competency: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Attitude: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Teamwork: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Productivity: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  Leadership: number;

  @Column({ type: 'varchar', length: 5, nullable: true })
  Grade: string;

  @Column({ type: 'text', nullable: true })
  Strengths: string;

  @Column({ type: 'text', nullable: true })
  Weaknesses: string;

  @Column({ type: 'text', nullable: true })
  Goals: string;

  @Column({ type: 'varchar', length: 20, default: 'Draft' })
  Status: string;
}
```

Y nghia:

| Cot | Y nghia |
| --- | --- |
| `ReviewID` | Ma phieu danh gia |
| `EmployeeID` | Nhan vien duoc danh gia |
| `ReviewPeriod` | Ky danh gia |
| `ReviewDate` | Ngay danh gia |
| `ReviewerID` | Nguoi danh gia |
| `OverallScore` | Diem tong |
| `Competency` | Nang luc |
| `Attitude` | Thai do |
| `Teamwork` | Lam viec nhom |
| `Productivity` | Nang suat |
| `Leadership` | Lanh dao |
| `Grade` | Xep loai |
| `Strengths` | Diem manh |
| `Weaknesses` | Diem can cai thien |
| `Goals` | Muc tieu tiep theo |
| `Status` | Trang thai phieu |

## 4. Phan Tich Du Lieu HR

Phan nay nam chu yeu o:

- `frontend/src/pages/Reports.jsx`
- `frontend/src/utils/analytics.js`

### Reports.jsx

Trang `Reports.jsx` lay du lieu tu:

- `getEmployees()`
- `getPayroll()`
- `getAttendanceSummary()`

Sau do tao cac bao cao:

- Bao cao nhan su
- Bao cao luong
- Bao cao cham cong
- Bao cao tong hop

Trang nay co:

- Bo loc phong ban
- Chon loai bao cao
- Xuat Excel
- Xuat PDF
- Bieu do luong theo phong ban
- Bieu do phan bo nhan vien
- Bieu do xu huong luong

### analytics.js

File nay gom cac ham xu ly du lieu:

```js
export function enrichPayrollRows(payrollRows, employees)
```

Ghep du lieu luong voi du lieu nhan vien de co them `DepartmentID`, `DepartmentLabel`, `EmployeeStatus`.

```js
export function buildSalaryBreakdown(rows)
```

Tinh tong:

- Luong co ban
- Thuong
- Khau tru
- Thuc linh

```js
export function buildPayrollTrend(rows)
```

Gom nhom luong theo ngay/thang de ve xu huong.

```js
export function buildSalaryByDepartment(rows)
```

Tinh tong luong theo phong ban.

```js
export function buildEmployeeDistribution(employees)
```

Dem so nhan vien theo trang thai: Active, On Leave, Probation, Intern, Inactive...

```js
export function buildAlerts(payrollRows, attendanceSummary, threshold)
```

Phat hien bat thuong:

- Luong tang qua cao
- So ngay vang vuot nguong

## 5. Dashboard Quan Tri

File chinh:

```text
frontend/src/pages/Dashboard.jsx
```

Dashboard goi nhieu API song song:

```js
const [
  employeesResponse,
  attendanceResponse,
  payrollResponse,
  syncResponse,
  lifecycleResponse,
  leaveResponse,
  overtimeResponse,
  benefitsResponse,
  kpiResponse,
  adjustmentsResponse,
  backupResponse,
] = await Promise.all([...]);
```

Rieng KPI duoc lay bang:

```js
canReadKpi ? listRecords('/kpi-okr', { limit: 100 }) : Promise.resolve({ data: [] })
```

Sau do tinh KPI trung binh:

```js
const averageKpi =
  state.srs.kpi?.length > 0
    ? state.srs.kpi.reduce((sum, item) => sum + Number(item.Score ?? 0), 0) / state.srs.kpi.length
    : 0;
```

Dashboard hien thi cac card:

- Tong so nhan vien
- Tong ban ghi cham cong
- Tong luong
- Nhan vien dang lam viec
- Ho so vong doi
- Nghi phep / tang ca
- Chi phi phuc loi
- KPI trung binh
- Trang thai API
- Thoi gian cap nhat gan nhat

## 6. CRUD Dung Chung Hoat Dong Nhu The Nao

File:

```text
backend/src/modules/crud/crud.service.ts
```

Day la service dung chung cho nhieu module. Cac service nhu `KpiOkrService` va `PerformanceEvaluationService` ke thua no.

### findAll

```ts
async findAll(query, actor)
```

Chuc nang:

- Phan trang bang `page`, `limit`
- Tim kiem bang `search`
- Sap xep theo `defaultOrder`
- Neu user la employee va entity co `EmployeeID`, tu dong loc du lieu cua chinh nhan vien do

### create

```ts
async create(body, actor, ipAddress)
```

Chuc nang:

- Tao record moi bang TypeORM repository
- Ghi audit log hanh dong `CREATE`
- Tra response thanh cong

### update

```ts
async update(id, body, actor, ipAddress)
```

Chuc nang:

- Tim record cu
- Cap nhat bang `Object.assign`
- Ghi audit log hanh dong `UPDATE`

### remove

```ts
async remove(id, actor, ipAddress)
```

Chuc nang:

- Neu service co khai bao `softDeleteField`, thi xoa mem bang cach doi trang thai
- Neu khong co `softDeleteField`, thi xoa that trong database
- Ghi audit log hanh dong `DELETE`

Vi du KPI khai bao:

```ts
softDeleteField: 'Status',
softDeleteValue: 'Cancelled',
```

Nen khi xoa KPI, record khong bi mat han, ma `Status` doi thanh `Cancelled`.

## 7. Phan Quyen

Frontend phan quyen trong:

```text
frontend/src/utils/accessControl.js
```

Route:

```js
'/kpi-okr': { roles: ['ADMIN', 'HR_MANAGER', 'EMPLOYEE'] },
'/performance-evaluation': { roles: ['ADMIN', 'HR_MANAGER', 'EMPLOYEE'] },
```

Endpoint:

```js
'/kpi-okr': { roles: ['ADMIN', 'HR_MANAGER', 'EMPLOYEE'] },
'/performance-evaluation': { roles: ['ADMIN', 'HR_MANAGER', 'EMPLOYEE'] },
```

Backend phan quyen bang decorator:

```ts
@Permissions('kpi.manage')
@Permissions('performance.manage')
```

Nghia la backend yeu cau user co permission tuong ung moi duoc goi API.

## 8. Luong Du Lieu Tong Quan

### KPI / OKR

```text
User mo /kpi-okr
  -> React Router render ManagementPage voi config kpiOkr
  -> ManagementPage goi listRecords('/kpi-okr')
  -> srsService.js goi apiClient.get('/kpi-okr')
  -> NestJS KpiOkrController nhan request
  -> KpiOkrService goi CrudService.findAll()
  -> TypeORM query bang kpi_okr
  -> Tra data ve frontend
  -> DataTable hien thi danh sach KPI
```

### Danh Gia Hieu Suat

```text
User mo /performance-evaluation
  -> React Router render ManagementPage voi config performanceEvaluation
  -> ManagementPage goi listRecords('/performance-evaluation')
  -> Backend PerformanceEvaluationController nhan request
  -> PerformanceEvaluationService / CrudService xu ly
  -> TypeORM query bang performance_reviews
  -> Tra danh sach phieu danh gia ve frontend
```

### Tu Dong Tinh Danh Gia

```text
POST /performance-evaluation/auto-calculate
  -> Kiem tra EmployeeID va ReviewPeriod
  -> Parse ky danh gia
  -> Tinh diem cham cong tu bang attendance
  -> Tinh diem KPI tu bang kpi_okr
  -> Diem tong = KPI 60% + cham cong 40%
  -> Quy doi diem sang thang 5
  -> Xep loai A+, A, B+, B, C, D
  -> Luu vao bang performance_reviews
```

## 9. Lien Ket KPI Voi Tinh Luong

KPI con duoc su dung trong module tinh luong:

```text
backend/src/modules/payroll/payroll.service.ts
```

Trong ham `computePayrollComponents`, backend lay KPI da duyet:

```ts
const kpiRows = await this.kpiRepo
  .createQueryBuilder('kpi')
  .where('kpi.EmployeeID = :employeeId', { employeeId })
  .andWhere("kpi.PeriodType = 'Monthly'")
  .andWhere("kpi.Status = 'Approved'")
  .getMany();

const kpiBonus = kpiRows.reduce(
  (sum, row) => sum + Number(row.BonusAmount ?? 0),
  0,
);
```

Y nghia:

- Chi KPI co `Status = Approved` moi duoc cong vao luong
- Chi KPI theo thang `PeriodType = Monthly` moi duoc tinh trong ky luong
- Tong `BonusAmount` cua KPI se cong vao thanh phan thuong KPI

## 10. Doi Chieu Voi So Do

| Muc trong so do | File/code tuong ung | Chuc nang |
| --- | --- | --- |
| Hieu suat & Phan tich nhan su | `Navbar.jsx`, `Dashboard.jsx`, `Reports.jsx`, `srs-features` | Nhom chuc nang quan ly hieu suat va bao cao |
| KPI / OKR | `managementConfigs.kpiOkr`, `KpiOkrController`, `KpiOkrService`, `KpiOkr` entity | Tao/sua/xoa/xem KPI, diem, trong so, thuong |
| Danh gia hieu suat | `managementConfigs.performanceEvaluation`, `PerformanceEvaluationController`, `PerformanceEvaluationService`, `PerformanceReview` entity | Quan ly phieu danh gia, tinh diem tu KPI va cham cong |
| Phan tich du lieu HR | `Reports.jsx`, `analytics.js` | Bao cao nhan su, luong, cham cong, bieu do, export |
| Dashboard quan tri | `Dashboard.jsx` | Tong quan KPI, nhan su, luong, cham cong, API status |

## 11. Nhan Xet Nhanh

Phan da implement ro:

- KPI / OKR CRUD
- Danh gia hieu suat CRUD
- Auto-calculate danh gia hieu suat tu KPI va cham cong
- Dashboard co KPI trung binh
- Reports co phan tich nhan su/luong/cham cong
- KPI co lien ket sang tinh luong qua `BonusAmount`

Phan trong tai lieu co nhac nhung chua thay code rieng ro rang:

- Thiet lap ky danh gia rieng
- Feedback 360 do
- Dashboard chuyen sau rieng cho performance analytics

Co the noi phan cua ban hien duoc cau truc theo huong:

```text
Frontend config-driven CRUD
  -> Backend generic CRUD
  -> TypeORM entity
  -> MySQL table
  -> Dashboard/Reports doc va tong hop du lieu
```


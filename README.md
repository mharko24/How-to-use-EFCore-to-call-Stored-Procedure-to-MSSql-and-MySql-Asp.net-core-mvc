# How-to-use-EFCore-to-call-Stored-Procedure-to-MSSql-and-MySql-Asp.net-core-mvc
How to use EFCore to call Stored Procedure
////////////////////////////// in controller
var genReq = _accSubContext.GeneralReqData(AsOfDate, ppl.ProjectCode).ToList(),
var goodIssue = _accSubContext.GoodIssuesData(AsOfDate, ppl.ProjectCode).ToList(),
var inventoryItem = _accSubContext.InventoryItemsData(AsOfDate, ppl.ProjectCode).ToList(),
var nonInventoryItems = _accSubContext.NonInventoryItemsData(AsOfDate, ppl.ProjectCode).ToList(),
var manpowerSupervisory = _payContext.ManpowerSupervisoryData(AsOfDate, ppl.ProjectCode).ToList(),
var manpowerDirect = _payContext.ManpowerDirectData(AsOfDate, ppl.ProjectCode).ToList(),


/////////////////////////////////////////////////////////////// Program.cs
var builder = WebApplication.CreateBuilder(args);
var payCon = builder.Configuration.GetConnectionString("PayFactorConnection");

var AccountsubCon = builder.Configuration.GetConnectionString("SubconConnection");
var serverVersion = new MySqlServerVersion(new Version(10, 4, 28));

// Add services to the container.
builder.Services.AddControllersWithViews();

builder.Services.AddMvc();
builder.Services.AddDbContext<PayFactorDbContext>(options =>
{
    options.UseMySql(payCon, serverVersion, msOptions =>
    {
        msOptions.EnableRetryOnFailure();
    })
    .LogTo(Console.WriteLine, LogLevel.Information)
    .EnableSensitiveDataLogging()
    .EnableDetailedErrors()
    .ConfigureWarnings(warnings =>
    {
        warnings.Throw(RelationalEventId.ConnectionError);
    });
}, ServiceLifetime.Scoped);

builder.Services.AddDbContext<AccountSubconDbContext>(options =>
{
    options.UseSqlServer(AccountsubCon);
}, ServiceLifetime.Scoped);


////////////////////////////////////////////////////////////////////  dbcontext in MSSQL
using DTR_Management_System.External.Entities.ASEC_Subcon;
using DTR_Management_System.External.Entities.AuthUser;
using Microsoft.Data.SqlClient;
using Microsoft.EntityFrameworkCore;

namespace DTR_Management_System.Data
{
    public class AccountSubconDbContext : DbContext
    {
        public AccountSubconDbContext(DbContextOptions<AccountSubconDbContext> options) : base(options)
        {
        }
        public virtual DbSet<APV_GenReq> GenReqs { get; set; }
        public virtual DbSet<PPC_GoodsIssue> GoodIssues { get; set; }
        public virtual DbSet<PPC_InventoryItem> InventoryItems { get; set; }
        public virtual DbSet<PPC_NonInventoryItem> NonInventoryItems { get; set; }


        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<APV_GenReq>(x =>
            {
                x.HasNoKey();
            });
            modelBuilder.Entity<PPC_GoodsIssue>(x =>
            {
                x.HasNoKey();
            });
            modelBuilder.Entity<PPC_InventoryItem>(x =>
            {
                x.HasNoKey();
            });
            modelBuilder.Entity<PPC_NonInventoryItem>(x =>
            {
                x.HasNoKey();
            });
        }
        public List<APV_GenReq> GeneralReqData(DateTime? AsOfDate, string? ProjectCode)
        {
            var results = Set<APV_GenReq>().FromSqlRaw("ASEC_PPC_PnL_GenReqAPVOutgoing  @DateTo, @PrjCode",
                                   new SqlParameter("@DateTo", AsOfDate),
                                   new SqlParameter("@PrjCode",ProjectCode))
                .ToList();
            return results;
        }
        public List<PPC_GoodsIssue> GoodIssuesData (DateTime? AsOfDate, string? ProjectCode)
        {
            var results = Set<PPC_GoodsIssue>().FromSqlRaw("ASEC_PPC_PnL_GoodsIssue  @DateTo, @PrjCode",
                                    new SqlParameter("@DateTo", AsOfDate),
                                    new SqlParameter("@PrjCode", ProjectCode))
                .ToList();
            return results;
        }
        public List<PPC_InventoryItem> InventoryItemsData(DateTime? AsOfDate, string? ProjectCode)
        {
            var results = Set<PPC_InventoryItem>().FromSqlRaw("ASEC_PPC_PnL_InventoryItem  @DateTo, @PrjCode",
                                   new SqlParameter("@DateTo", AsOfDate),
                                   new SqlParameter("@PrjCode", ProjectCode))
                .ToList();
                return results;
        }
        public List<PPC_NonInventoryItem> NonInventoryItemsData(DateTime? AsOfDate, string? ProjectCode)
        {
            var results = Set<PPC_NonInventoryItem>().FromSqlRaw("ASEC_PPC_PnL_NonInventoryItem  @DateTo, @PrjCode",
                                   new SqlParameter("@DateTo", AsOfDate),
                                   new SqlParameter("@PrjCode", ProjectCode))
                .ToList();
            return results;
        }
    }
  
}
//////////////////////////////////////////// dbContext in MySql

using DTR_Management_System.External.Entities.ASEC_PayFactor;
using DTR_Management_System.External.Entities.PayFactor;
using Microsoft.EntityFrameworkCore;
using MySqlConnector;

namespace DTR_Management_System.Data
{
    public class PayFactorDbContext : DbContext
    {
        public PayFactorDbContext(DbContextOptions<PayFactorDbContext> options) : base(options)
        {
        }
        public virtual DbSet<posteddtr> Posteddtrs { get; set; }
        public virtual DbSet<PPC_ManpowerSupervisory> ManpowerSupervisories { get; set; }
        public virtual DbSet<PPC_ManpowerDirect> ManpowerDirects { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<posteddtr>(x =>
            {
                x.HasNoKey();
                x.ToView("posteddtr");
            });
            modelBuilder.Entity<PPC_ManpowerSupervisory>().HasNoKey();
            modelBuilder.Entity<PPC_ManpowerDirect>().HasNoKey();

        }
        public virtual List<PPC_ManpowerSupervisory> ManpowerSupervisoryData(DateTime? AsofDate, string? ProjCode)
        {
            object[] parameters =
            {
                new MySqlParameter("@DateTo", AsofDate),
                new MySqlParameter("@PrjCode", ProjCode),

            };
            var result = Set<PPC_ManpowerSupervisory>().FromSqlRaw("CALL ASEC_SP_PPC_PnL_ManpowerSupervisory  (@DateTo, @PrjCode)", parameters).ToList();
            return result;
        }
        public List<PPC_ManpowerDirect> ManpowerDirectData(DateTime? AsofDate, string? PrjCode)
        {
            object[] parameters ={
                new MySqlParameter ("@DateTo", AsofDate),
                new MySqlParameter ("@PrjCode", PrjCode)
            };
            var result = Set<PPC_ManpowerDirect>().FromSqlRaw("CALL ASEC_SP_PPC_PnL_ManpowerDirect (@DateTo,@PrjCode)", parameters).ToList();
            return result;
        }
    }
}

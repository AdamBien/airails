# Noise Words

Pattern vocabulary to strip from compound names before concept clustering. Stripping exposes the domain nucleus: `AbstractCustomerContractValidationServiceFactoryImpl` → Customer, Contract, Validation.

**Context rule:** these words are noise in *code identifiers*. If one appears in a database column, table, or UI label, treat it as a domain concept (Booking in payments, Position in trading, Service in telco) and record the collision in the Noise Report.

## Structural prefixes

Abstract, Base, Default, Generic, Simple, Basic, Common, Core, Internal, My

## Pattern suffixes

Impl, Manager, Service, Facade, Factory, Builder, Helper, Util, Utils, Handler, Provider, Processor, Executor, Dispatcher, Resolver, Registry, Locator, Broker, Mediator, Coordinator, Orchestrator, Strategy, Template, Command, Visitor, Observer, Listener, Interceptor, Decorator, Wrapper, Adapter, Proxy, Delegate, Bridge, Gateway, Repository, DAO, DTO, VO, BO, PO, TO, Bean, Entity*, Model, Type, Info, Data, Object, Instance, Context, Session, Config, Configuration, Settings, Properties, Constants, Enum, Exception, Error, Fault, Controller, Servlet, Action, Form, View, Page, Resource*, Endpoint, Client, Stub, Skeleton, Home, Remote, Local

`*` — strip only as suffix of a longer name (`OrderEntity` → Order); standalone occurrences may be domain terms.

## Technical package segments

impl, util, common, core, internal, api, spi, ejb, web, ui, gui, persistence, dao, dto, model, service, business, logic, framework, base, shared, misc, helper

## Abbreviation expansion hints

Legacy schemas abbreviate aggressively (`CNTRCT`, `VTRG`, `CUST_NO`). Expand from context before clustering: matching column comments, i18n labels for the same field, or Javadoc of the mapped entity. Record unexpandable abbreviations as low-confidence concepts under Open Questions.
